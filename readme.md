My journey to get secure boot running on my HP Proliant Server.

Symptom: as soon as secure boot was enabled, the server couldn't boot up anymore

TL;DR: you have to import the HP certificate for the RAID drivers too.

Details:
All files were signed correctly, the automatic deployment via sbctl and with `--yes-this-might-brick-my-machine` worked like a charm, but the verification still failed.

My first assumption (based on this support PDF) was, that the key size of 4096 bit keys is too large: https://support.hpe.com/hpesc/public/docDisplay?docId=a00068482en_us&docLocale=en_US

This `--yes-this-might-brick-my-machine` parameter braught the final resolution, as a few manufacturers sign their drivers (used by UEFI) itself with by a Microsoft CA to initialize the Dispaly.
What I realized meanwhile was, that always when enforcing secure boot, my local raid disk disappeaered.

So basically my theory was, that the UEFI itself cannot read the RAID content anymore.
I tested sbctl enroll-keys with parameter -m for "Microsoft Certficiate Authority", as an additional accepted paramter. And here we are, there we go, from now on, the Network Boot Stack was able to load again. But my RAID was still not there.

So I reenabled the factory platform keys and performed an export of the original keys with:

`efi-readvar -v PK -o 'PK'`

`efi-readvar -v KEK -o 'KEK'`

`efi-readvar -v db -o 'db'`

`efi-readvar -v dbx -o 'dbx'`

The links behind these exports are of base64 encoded content. Original export files are binary.
They were extracted from a UEFI Firmware Update dated 08/2024.

and with a bit AI stuff I could find out, that the first certificate in the db chain, was the "HP UEFI Secure Boot 2013 DB Key" so I isolated this one:
```
-----BEGIN CERTIFICATE-----
MIIFeDCCBGCgAwIBAgIQVnSnA+85CRCLH0dTaHNtbTANBgkqhkiG9w0BAQsFADBr
MQswCQYDVQQGEwJVUzEgMB4GA1UEChMXSGV3bGV0dC1QYWNrYXJkIENvbXBhbnkx
OjA4BgNVBAMTMUhld2xldHQtUGFja2FyZCBQcmludGluZyBEZXZpY2UgSW5mcmFz
dHJ1Y3R1cmUgQ0EwHhcNMTMwODIzMDAwMDAwWhcNMzMwODIzMjM1OTU5WjB5MSAw
HgYDVQQKExdIZXdsZXR0LVBhY2thcmQgQ29tcGFueTErMCkGA1UECxQiTG9uZyBM
aXZlZCBDb2RlU2lnbmluZyBDZXJ0aWZpY2F0ZTEoMCYGA1UEAxQfSFAgVUVGSSBT
ZWN1cmUgQm9vdCAyMDEzIERCIGtleTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
AQoCggEBAMlHv4hkphZZV29sfwzKCN+XgqE77zYCFUOb6pqr5U28+7S0T4WJ5Kfl
JdTmmzZLmY4vAX3hkNiKOWGOkGafc9PzOC5o/eYY2/M+rITsZBzpB6Py51JrKHVv
NHmrJD+nzP38OjJx/LMiERhgdusxoiiNcvM8r02VAb2i8LFDal0UMzsRcoxpcau0
ox3HM2SGin/hQTPYGRWacRTfunYQyY8D/Vw6TMWT5am2QOADe3VHejo+OlGv00Mq
K8pqz9A8RB7pv8mvprPQxMDzGLmcpmtuPFdOgvZqRDJl2jhS9FufvLEiGpjz8OGJ
paGLpJohMSqJtevzsZtEk9UCtaUrFosCAwEAAaOCAggwggIEMAwGA1UdEwEB/wQC
MAAwawYDVR0fBGQwYjBgoF6gXIZaaHR0cDovL29uc2l0ZWNybC52ZXJpc2lnbi5j
b20vSGV3bGV0dFBhY2thcmRDb21wYW55RGVTUHJpbnRpbmdEZXZpY2VDU0lEVGVt
cC9MYXRlc3RDUkwuY3JsMA4GA1UdDwEB/wQEAwIHgDCB4QYDVR0gBIHZMIHWMIHT
BgorBgEEAQsEBAEBMIHEMIHBBggrBgEFBQcCAjCBtBqBsUhld2xldHQgUGFja2Fy
ZCBDb21wYW55LCAyLCBBdXRob3JpdHkgdG8gYmluZCBIZXdsZXR0LVBhY2thcmQg
Q29tcGFueSBkb2VzIG5vdCBjb3JyZXNwb25kIHdpdGggdXNlIG9yIHBvc3Nlc3Np
b24gb2YgdGhpcyBjZXJ0aWZpY2F0ZS4gSXNzdWVkIHRvIGZhY2lsaXRhdGUgY29t
bXVuaWNhdGlvbiB3aXRoIEhQLjA7BggrBgEFBQcBAQQvMC0wKwYIKwYBBQUHMAGG
H2h0dHA6Ly9vbnNpdGUtb2NzcC52ZXJpc2lnbi5jb20wHQYDVR0OBBYEFB188sK5
JnP2nI7h7HBjlnq5tivsMB8GA1UdIwQYMBaAFLihDL0GX0YR6YDb95m9HfT96g3G
MBYGA1UdJQEB/wQMMAoGCCsGAQUFBwMDMA0GCSqGSIb3DQEBCwUAA4IBAQBFzx6B
vBJCurNC31IQ4ilrLmlmM4L0pEfKc3hUA+f+7eWH5225BOsI/yE6MsOcQgRSKj+X
8CvVvJN7QT456SdnefLaOkGBmpmdoXnhxpRr8Jgd7duURJvSPMvI0zVH16iWx9ON
P6TpbTTsqWxoWCOfJ2o25uKamOGSprKwbinsk53SNVmmBquDvs/Dfl34Etm6Z3Z6
meEYBVH6NIXrH4VfGaoyvTJBmHIiaSRBIsusrnMXF1pZKNcp+Kf9VI7PvhTrRh6C
jXeNk82TBptW2rKyTsmWiG2XBHKI4/eoHqA3r9xuBa7oyFAoWkGj6krtfgyt/G6A
68k2VJenVGEMNGBA
-----END CERTIFICATE-----
```

The challenge is now, that as soon as you are importing the Platform Key, your Setup is completed, so you have to take care about a strict procedure how to import the certificates:
- create a fresh set of custom secure boot certificates: sbctl create-keys
- place the HP certificate under `/var/lib/sbctl/keys/custom/db/hp.pem`
- enroll only db keys, together with the custom keys defined: `sbctl enroll-keys --partial db --yes-this-might-brick-my-machine -c`
- enroll the KEK key: `sbctl enroll-keys --partial KEK --yes-this-might-brick-my-machine`
- optional: enroll the dbx: `efi-updatevar -e -k /var/lib/sbctl/keys/KEK/KEK.key -f /root/dbx dbx` (notes; no signatures by else then 77fa9abd-0359-4d32-bd60-28f4e78f784b and the CRL from HP has no revocations as of today (June 2025))
- enroll the PK key: `sbctl enroll-keys --partial PK --yes-this-might-brick-my-machine`

in case you get:
```
# sbctl enroll-keys --partial KEK --yes-this-might-brick-my-machine
‼ File is immutable: /sys/firmware/efi/efivars/db-d719b2cb-3d3a-4596-a3bc-dad00e67656f
You need to chattr -i files in efivarfs
```
```
# sbctl enroll-keys --partial PK --yes-this-might-brick-my-machine
‼ File is immutable: /sys/firmware/efi/efivars/KEK-8be4df61-93ca-11d2-aa0d-00e098032b8c
You need to chattr -i files in efivarfs
```
you indeed need to change the attributes:
```
chattr -i /sys/firmware/efi/efivars/db-*
```
```
chattr -i /sys/firmware/efi/efivars/KEK-*
```

Your efi-readvar output should now look like this:
```
# efi-readvar
Variable PK, length 1301
PK: List 0, type X509
    Signature 0, size 1273, owner a60eb048-117c-4f4e-9d3a-a57b46b9b4a4
        Subject:
            C=Platform Key, CN=Platform Key
        Issuer:
            C=Platform Key, CN=Platform Key
Variable KEK, length 1316
KEK: List 0, type X509
    Signature 0, size 1288, owner a60eb048-117c-4f4e-9d3a-a57b46b9b4a4
        Subject:
            C=Key Exchange Key, CN=Key Exchange Key
        Issuer:
            C=Key Exchange Key, CN=Key Exchange Key
Variable db, length 2749
db: List 0, type X509
    Signature 0, size 1273, owner a60eb048-117c-4f4e-9d3a-a57b46b9b4a4
        Subject:
            C=Database Key, CN=Database Key
        Issuer:
            C=Database Key, CN=Database Key
db: List 1, type X509
    Signature 0, size 1420, owner 88a69775-5ad7-45d9-9f34-cec43e1f1989
        Subject:
            O=Hewlett-Packard Company, OU=Long Lived CodeSigning Certificate, CN=HP UEFI Secure Boot 2013 DB key
        Issuer:
            C=US, O=Hewlett-Packard Company, CN=Hewlett-Packard Printing Device Infrastructure CA
Variable dbx has no entries
Variable MokList has no entries
```

and your sbctl status is now looking like this:
```
 sbctl status
Installed:      ✓ sbctl is installed
Owner GUID:     a60eb048-117c-4f4e-9d3a-a57b46b9b4a4
Setup Mode:     ✓ Disabled
Secure Boot:    ✗ Disabled
Vendor Keys:    custom
```

also make sure your files are correctly signed:
```
# sbctl verify
Verifying file database and EFI images in /boot...
✓ /boot/EFI/BOOT/BOOTX64.EFI is signed
✓ /boot/EFI/Linux/arch-linux.efi is signed
✓ /boot/EFI/systemd/systemd-bootx64.efi is signed
✓ /boot/vmlinuz-linux is signed
```

4096 bit is btw no problem :)
