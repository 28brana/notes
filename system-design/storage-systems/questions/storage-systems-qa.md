# Q&A · Storage Systems

[← Back to Storage Systems](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Block Storage

[Read the notes →](../01-block-storage.md)

> [!question]- 🟢 Q1. What is the difference between block, file, and object storage?
> Block storage is a raw disk split into blocks, mounted by one server — fastest, used for databases and OS disks. File storage is a shared folder many servers mount over the network, like NFS or EFS. Object storage is a flat key-value store over HTTP, like S3 — cheapest and almost unlimited, great for photos and backups, but no in-place edits.

> [!question]- 🟡 Q2. What are IOPS, and why do they matter for a database?
> IOPS is the number of read/write operations a disk can do per second. Databases do lots of small random reads and writes, so they hit the IOPS limit long before running out of space or MB/s. If IOPS are too low, queries queue up and latency jumps even when CPU looks idle.

> [!question]- 🔴 Q3. Your database VM crashed. Is the data lost, and what if the whole availability zone goes down?
> If the data is on a network block volume like EBS, it survives the VM — you attach the volume to a new VM and start the database again. But EBS is replicated only inside one AZ, so an AZ outage makes it unreachable. To survive that, you need a replica in another AZ or snapshots copied to object storage, depending on your RPO and RTO.

---

## 02 · Distributed File Systems

[Read the notes →](../02-distributed-file-systems.md)

> [!question]- 🟢 Q4. How does a distributed file system like HDFS store a file?
> It splits the file into large chunks, around 128 MB, and stores three replicas of each chunk on different machines and racks. A NameNode keeps only the metadata — which chunks make up a file and where they live. Clients ask the NameNode for locations, then read directly from the DataNodes.

> [!question]- 🔴 Q5. Your HDFS cluster is running out of NameNode memory, but the disks are half empty. What's wrong?
> Most likely there are millions of tiny files. The NameNode keeps metadata for every file and chunk in memory, so lots of small files fill it up even though they use little disk. The fix is to combine small files into bigger ones, or move that data to object storage.

---

## 03 · Object Storage

[Read the notes →](../03-object-storage.md)

> [!question]- 🟢 Q6. What is object storage, and what is it good for?
> Object storage keeps files as objects — bytes plus metadata — in a bucket, each found by a unique key, over a simple HTTP API. The namespace is flat, so it scales to trillions of objects and is very durable and cheap. It's ideal for photos, videos, backups, and logs, but not for databases or files you edit in place.

> [!question]- 🟡 Q7. How would you design photo upload for a social media app?
> The client asks the app server for a presigned URL, a short-lived signed link for one specific key. The client uploads the photo straight to S3, so the bytes never pass through our servers. We store only the key and metadata in the database, an upload event triggers a worker to make thumbnails, and a CDN serves the photos to users.

> [!question]- 🔴 Q8. Your storage bill grows 20% every month, but most photos are never viewed after a week. What do you do?
> Add lifecycle rules: move objects to an infrequent-access tier after about 30 days and to an archive tier after a year. Also delete leftovers like failed uploads and old temp files automatically. Hot recent photos stay in the standard tier, served through a CDN, so users don't notice.

---

## 04 · Erasure Coding

[Read the notes →](../04-erasure-coding.md)

> [!question]- 🟢 Q9. What is erasure coding and how does it compare to replication?
> Erasure coding splits data into k data pieces and adds m parity pieces, and any k pieces can rebuild the data. With 6+3 you survive three lost pieces using only 1.5x storage, while 3x replication uses 3x storage and survives two failures. So it's much cheaper for the same durability.

> [!question]- 🔴 Q10. If erasure coding is so much cheaper, why not use it for everything?
> Reads and rebuilds are expensive. A read may need pieces from several servers, and rebuilding one lost piece means reading k pieces over the network and doing CPU work. So hot, small, latency-sensitive data usually stays replicated, and erasure coding is used for large or cold data.
