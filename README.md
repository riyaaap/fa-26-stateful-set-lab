# Deployment vs StatefulSet

First double check that your cluster has a storage class. If this does not show anything, then please ask on Ed for help.

```bash
kubectl get storageclass
```

## 1. Deployment

```bash
kubectl apply -f deployment.yml
kubectl get pods -l app=note
kubectl get pvc
```

The pods stay `Pending` for a few seconds while the volume is created. Run the `get pods` command again until both rows say `Running` before you answer, and before you write a note.

**Q1.** What do you observe about the pods?
Both pods have the same age, and seem like they might be two separate instances/replicas of the same pod since they have almost identical names except for the suffix/last 5-character code that might just be added to the name everytime a unique pod instance is created. 
Also when we do get pvc, there is only one pvc, so the two pods are sharing one volume. 

## 2. Write on one pod, read on the other

Copy the two pod names from the previous command. Run these one at a time, with those names filled in:

```bash
kubectl exec POD_A -- sh -c 'echo hello-from-a > /data/note.txt'
kubectl exec POD_B -- cat /data/note.txt
kubectl get pvc
```

**Q2.** What do you observe about the note, and about the volumes?
Even when we write the note from one pod, we are able to read it from the other pod.. this indicates that they are both sharing the same volume.

## 3. Remove the Deployment

```bash
kubectl delete -f deployment.yml
kubectl get pods -l app=note
kubectl get pvc
```

Wait until the pods and the `data-note` claim are gone before continuing.

## 4. StatefulSet

Run these two commands back to back. `-w` keeps printing pod updates until you press Ctrl-C. If you wait too long to start it, both pods will already be `Running` and you will miss the order.

```bash
kubectl apply -f stateful.yml
kubectl get pods -l app=note -w
```

**Q3.** In what order did the pods start, and what are they named?

The pods were started in the order 0, then 1, and were named note-0 and note-1. 

## 5. A different note on each pod

```bash
kubectl exec note-0 -- sh -c 'echo hello-from-0 > /data/note.txt'
kubectl exec note-1 -- sh -c 'echo hello-from-1 > /data/note.txt'
kubectl exec note-0 -- cat /data/note.txt
kubectl exec note-1 -- cat /data/note.txt
kubectl get pvc
```

**Q4.** How does this differ from what you saw in Q2?
It created two new pods and printed both out hello-from-0 and hello-from-1 in order, and created two separate pvc's this time with two separate volumes, with the pvc's named as data-note-0 and data-note-1. 

## 6. Delete note-0

```bash
kubectl delete pod note-0
kubectl get pods -l app=note -w
```

Wait until `note-0` is `Running` again, then press Ctrl-C.

The pod that is shutting down can show `Error` for a moment. That is the old container exiting. The replacement is the `note-0`.

```bash
kubectl exec note-0 -- cat /data/note.txt
kubectl exec note-1 -- cat /data/note.txt
kubectl get pvc
```

**Q5.** After `note-0` was deleted and came back, what stayed the same? How do these volumes differ from the Deployment?

Even after note-0 was deleted and came back, the volume and pvc names/hashcode/identifiers still stayed the exact same. 

## Cleanup

```bash
kubectl delete -f stateful.yml
kubectl delete pvc data-note-0 data-note-1
```

(Note: Deleting the StatefulSet alone does not delete its volumes)
