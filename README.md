# Kubernetes-PV-PVC

first step we are create the cluster
after that we are going to vscode
we are connecting that kubernetes cluster

# ek storage account banaya aur uske andar file share create kiya.

# firstly we are creating the secret.yaml file
# Inside the secret.yaml file we are write the code are given below

apiVersion: v1      # this is given in the pod.txt file
kind: Secret        # ye type likha secret 
metadata:           # metadata matlab data about data
  name: my-secret   # name likha hai secret ka
type: Opaque        # Opaque ek generic secret type hai, jo custom key-value pair store karne ke liye use hota hai
stringData:          # stringData use kiya gaya hai taki values plain text mein likh sake, kubernetes ise internally base64 encode karega
  azurestorageaccountname: bambhole #ye storage accountka naam diya hai
  azurestorageaccountkey: erjksdglsdjsjgsgsgkdgsgjlkjdlkbndlbelbln====EKMKRKM===
  # ye storage accountka key diya hai
  # ye dono cheezein secret.yaml mein hai, jo ki pod.yaml aur pv.yaml mein use hoti hain
  
  # fir apun ne secret ko create kar liya : kubectl apply -f secret.yaml
  # kubectl get secret.
------------------------------------------------------------------------------------------------

# after creation of secret , now we are creating the pv
we create the pv.yaml file
# for pv.txt creation we are just give the command in our terminal (kubectl explain pv --recursive > pv.txt)

now we are seen the pv.txt file, with help of the pv.txt file we are creating pv.yaml file

apiVersion: v1                # Kubernetes API version
kind: PersistentVolume        # Resource ka type PersistentVolume (PV) hai
metadata:                     
  name: my-pv                 # PV ka naam
spec:
  capacity:                  # PV ka storage capacity define kar rahe hain
    storage: 2Gi             # Storage ka size 2GiB diya hai
  accessModes:               # Access modes specify karte hain ki volume kaise use ho sakta hai
    - ReadWriteMany          # RWX: multiple pods ek hi samay read/write kar sakte hain
  persistentVolumeReclaimPolicy: Retain  # PVC delete hone ke baad bhi data delete nahi hoga
  storageClassName: my-storage-class     #(is PV ko bind karne ke liye PVC same storageClass ka liya)
  azureFile:                             # Azure File storage ka volume source
    secretName: my-secret    # Azure storage account credentials ka Secret ka naam (jo Secret.yaml me tha)
    shareName: pvc # ye share name diya hai jo file apni bani hui thi storage account ke andar uska nam

# now we are give the command for cretion of pv (kubectl apply -f .\pv.yaml)
# check the pv create or not (kubectl get pv)
---------------------------------------------------------------------------------------------------------

# now we are creating the pvc.yaml and pvc.txt file
# pvc.txt file are given below

apiVersion: v1                     # Kubernetes API version
kind: PersistentVolumeClaim         # Resource ka type PersistentVolumeClaim (PVC) hai
metadata:
  name: mypvc                       # PVC ka naam (pods is naam se claim karenge)
spec:
  resources:                        # Resource requirements specify kar rahe hain
    requests:
      storage: 1Gi                   # PVC 1Gi storage request kar raha hai
  accessModes:                       # Pod volume ko kaise access karega
    - ReadWriteMany                  # RWX: multiple pods ek hi samay read/write kar sakte hain
  storageClassName: my-storage-class # StorageClass specify kiya hai, isse PVC usi class wale PV ke sath bind hoga

# fir create kiya with using this command (kubectl apply -f .\pvc.yaml)
# check karte hai pvc banaya hai ya nahi (kubectl get pvc )
----------------------------------------------------------------------------------------------------------

# now we are creating pod.yaml with help of pod.txt file
# pod.yaml are given below

apiVersion: v1                 # Kubernetes API version
kind: Pod                      # Resource ka type Pod hai
metadata:
  name: myapp                  # Pod ka naam "myapp"
spec:
  containers:                  # Pod ke andar containers ka definition
  - name: myapp                # Container ka naam "myapp"
    image: nginx               # Container image Nginx use ho raha hai
    ports:                     # Container ke expose hone wale ports
      - containerPort: 80      # Port 80 expose kar raha hai (Nginx ka default HTTP port)
    volumeMounts:              # Container ke andar volumes mount karne ke liye
    - name: pendrive           # Volume ka naam "pendrive" reference kar rahe hain
      mountPath: /mnt/pendrive # Container ke andar mount point (yaha pe Azure File share mount hoga)
  volumes:                     # Pod ke liye volume definition
  - name: pendrive             # Volume ka naam "pendrive" (same as volumeMounts se match karega)
    persistentVolumeClaim:     # Volume ko ek PVC ke through attach karna hai
      claimName: mypvc         # PVC ka naam (mypvc) reference kiya, jo PV ke sath bind hota hai

# ye bhi create karenge (kubectl apply -f .\pod.yaml)
# check karenge apna pod bana hai ya nahi  (kubectl get pod)
# now we are enter inside the pod ( kubectl exec pod/<pod ka nam> -it -- bash )
# jo path mount kiya tha mnt/pendrive 
# cd mnt/pendrive
# ls
# mnt/pendrive ke andar apun check karne ke liye file banayega test.txt
# ye test.txt file hame staorage account ke andar dikh jayegi iska matlab apna pv pvc work kar raha hai
# apn pod bhi delete karenge tab bhi ye file pod delete hone ke baad bhi delete nahi hogi
# yahi pv pvc ka benefit tha jab pod bhi delete ho jaye aur naya pod create kare to file apne ko pod mai mil jaati hai
----------------------------------------------------------------------------------------------------------
