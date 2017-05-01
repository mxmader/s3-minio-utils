# s3-minio-utils
Bunch of utils aimed at ingesting data from various storage platforms like Google Drive and wrapping an S3-ish interface around that data using Minio

```
#!/bin/bash

# run as root

useradd s3
mkdir /opt/s3_root
chown /opt/s3_root s3:s3

wget -O /bin/gdrive "https://docs.google.com/uc?id=0B3X9GlR6EmbnQ0FtZmJJUXEyRTA&export=download"
chmod +x /bin/gdrive

wget -O /bin/minio "https://dl.minio.io/server/minio/release/linux-amd64/minio"
chmod +x /bin/minio

cat <<EOF > /usr/lib/systemd/system/minio-s3.service
[Unit]
Description=Minio S3 Server
After=network.target syslog.target
Requires=network.target

[Service]
Type=simple
User=s3
Group=s3
WorkingDirectory=/home/s3
ExecStart=/bin/minio server /opt/s3_root
StandardOutput=syslog
StandardError=syslog

[Install]
WantedBy=multi-user.target
EOF

systemctl enable minio-s3.service
systemctl start minio-s3.service

cat <<EOF > /home/s3/sync.sh
folder_id='google_drive_folder_id'
pushd /opt/s3_root
gdrive sync download ${folder_id} .
EOF

chmod +x /home/s3/sync.sh
chown s3:s3 /home/s3/sync.sh

# you're now ready to run a sync job
# s3 server will run at http://0.0.0.0:9000

```
