# duplicity-pydrive-backend

## Install dependecies
```
apt-get build-dep duplicity
apt-get install python-crypto python-setuptools checkinstall python-lockfile
```

## Generate gpg keys
```
gpg --gen-key
```
## Create service account in your Google account
- Go to [Google developers console](https://console.developers.google.com) and create a service account
- Write down am email address of the account, XXX@developer.gserviceaccount.com 
- Download a .p12 key file, then convert it to the .pem:
```
openssl pkcs12 -in XXX.p12  -nodes -nocerts > pydriveprivatekey.pem
```
(you'll need a password you got when created the account)

## Download, patch and install duplicity
```
wget duplicity-0.7.0.tar.gz
wget duplicity-0.7.0.tar.gz.sig 

gpg --verify duplicity-0.7.0.tar.gz.sig 
gpg: Signature made Thu 23 Oct 2014 04:19:49 PM IDT using DSA key ID E654E600
gpg: Can't check signature: public key not found

gpg --recv-keys E654E600
gpg: requesting key E654E600 from hkp server keys.gnupg.net
gpg: key E654E600: public key "Duplicity Test (no password) <duplicity@loafman.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1

gpg --verify duplicity-0.7.0.tar.gz.sig 
gpg: Signature made Thu 23 Oct 2014 04:19:49 PM IDT using DSA key ID E654E600
gpg: Good signature from "Duplicity Test (no password) <duplicity@loafman.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 9D95 920C ED4A 8D5F 8B08  6A9F 8B6F 8FF4 E654 E600

tar -xzvf duplicity-0.7.0.tar.gz
cd /duplicity-0.7.0
vi duplicity/backend.py
```
```
uses_netloc = ['ftp',
               'ftps',
               'hsi',
               's3',
               'scp', 'ssh', 'sftp',
               'webdav', 'webdavs',
               'gdocs',
               'http', 'https',
               'imap', 'imaps',
               'mega',
               'copy',
               'pydrive']

if pu.query:     
    try:
        self.keyfile = urlparse.parse_qs(pu.query)['keyfile']
    except Exception:
        raise InvalidBackendURL("Syntax error (keyfile) in: %s" % url_string)
```
```
mv pydrivebackend.py duplicity-0.7.0/duplicity/backends/
checkinstall python setup.py install
```
## Install PyDrive
```
apt-get install python-yaml python-simplejson python-pip
checkinstall pip install PyDrive
```
## Use the backend by this remote URL:
```
duplicity full --encrypt-key *YOUR_gpg_KEY* --sign-key *YOUR_gpg_KEY* /local/folder/path pydrive://XXX@developer.gserviceaccount.com/remote/path/on/google/drive?keyfile=/path/to/your/pydriveprivatekey.pem
```
