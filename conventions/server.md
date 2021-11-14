# Server conventions

## OS

I generally recommend Ubuntu Server. If it's only used as a docker host, Alpine Linux has proven to be lightweight, stable and secure.

Docker should be used as much as possible, as it provides flexibility and portability, especially in conjunction with docker-compose.

## Remote Storage

- NFSv4.1 should be preferred over NFS3, as file locking seems to work better. Not as good as native though. NFS3 with nolock mount option has proven to be unreliable and breaks especially database applications.
- NFS should be only used to serve large files, thus, packet size can be set to the maximum. (I use 64KB)
- iSCSI (file-based LUN, but block-level also works well) is ideal when handling lots of small files, especially database applications love iSCSI.
- iSCSI Queue depth is hardware dependent, but 128 seems to work fine
- If possible, remote storage should be used in a dedicated, isolated network.
