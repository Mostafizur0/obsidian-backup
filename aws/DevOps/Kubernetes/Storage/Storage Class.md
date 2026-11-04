[[Storage]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Lab-Solution-Storage-Class/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Storage-Class/page
![[Pasted image 20261104221926.png]]

- Static provisioning: <span style="color: #22C55E;">you create cloud disks ahead of time,</span> then create the matching PersistentVolume (PV) objects and PersistentVolumeClaims (PVCs).
- Dynamic provisioning (StorageClass): StorageClasses enable dynamic provisioning. <span style="color: #22C55E;">When a PVC is created that references a StorageClass, Kubernetes uses the provisioner defined in that StorageClass to create backing storage on the cloud provider. The provisioner creates the disk, a PV object is created automatically, and the PVC binds to that PV.</span>
![[Pasted image 20261104222311.png]]

Minimal StorageClass yaml
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```
Once a StorageClass exists you no longer need to create PVs manually. Instead, reference the StorageClass in your PVC with `storageClassName`:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["/bin/sh","-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      persistentVolumeClaim:
        claimName: myclaim
```

When you apply the PVC above, the StorageClass’s provisioner will create the disk in Google Cloud, create the corresponding PV, and bind the PVC to it. The PV still exists — it is just created automatically instead of by you.Quick comparison: static vs dynamic provisioning

|Provisioning type|Who creates cloud disk|Who creates PV object|When is disk created|
|---|---|---|---|
|Static|Operator / admin|Operator / admin|Before PVC is applied|
|Dynamic (StorageClass)|Provisioner (cloud driver)|Provisioner (Kubernetes)|When PVC is created and requests a StorageClass|
