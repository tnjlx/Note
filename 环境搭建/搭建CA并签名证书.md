- [1. ECC](#1-ecc)
  - [1.1. 建立CA](#11-建立ca)
  - [1.2. 签名证书](#12-签名证书)
- [2. RSA](#2-rsa)
  - [2.1. 建立CA](#21-建立ca)
  - [2.2. 签名证书](#22-签名证书)

# 1. ECC
## 1.1. 建立CA
openssl ecparam -genkey -name secp256k1 -out ca.key
openssl req -new -subj "/C=CN/ST=SC/L=CD/O=SM/OU=SM/CN=SMCA" -key ca.key -out ca.csr
openssl req -text -noout -verify -in ca.csr

echo "subjectKeyIdentifier=hash" > ca_cert_extensions
echo "authorityKeyIdentifier=keyid:always,issuer" >> ca_cert_extensions
echo "basicConstraints=critical,CA:true" >> ca_cert_extensions
echo "subjectAltName=IP:127.0.0.1" >> ca_cert_extensions

openssl x509 -req -days 3650 -sha256 -extfile ca_cert_extensions -signkey ca.key -in ca.csr -out ca.crt
openssl x509 -text -noout -in ca.crt

## 1.2. 签名证书
openssl ecparam -genkey -name secp256k1 -out ch002.key
openssl req -new -subj "/C=CN/ST=SC/L=CD/O=SM/OU=SM/CN=ch002" -key ch002.key -out ch002.csr
openssl req -text -noout -verify -in ch002.csr

echo "subjectKeyIdentifier = hash" > ch002_cert_extensions
echo "authorityKeyIdentifier = keyid:always,issuer" >> ch002_cert_extensions
echo "basicConstraints = CA:FALSE" >> ch002_cert_extensions
echo "keyUsage = nonRepudiation, digitalSignature, keyEncipherment" >> ch002_cert_extensions
echo "subjectAltName = IP:192.168.177.222" >> ch002_cert_extensions

openssl x509 -req -days 3650 -sha256 -extfile ch002_cert_extensions -CA ca.crt -CAkey ca.key -in ch002.csr -out ch002.crt -CAcreateserial
openssl x509 -text -noout -in ch002.crt

# 2. RSA
## 2.1. 建立CA
openssl genrsa -out ca.key 4096
openssl req -x509 -new -key ca.key -out ca.crt -days 3650 -subj "/C=CN/ST=SC/L=CD/O=SM/OU=SM/CN=SMCA"

## 2.2. 签名证书
openssl genrsa -out ch002.key 4096
openssl req -new -key ch002.key -out ch002.csr -subj "/C=CN/ST=SC/L=CD/O=SM/OU=SM/CN=ch002"
echo "subjectAltName=IP:192.168.177.222" > ch002_cert_extensions
openssl x509 -req -in ch002.csr -signkey ch002.key -out ch002.crt -days 3650 -extfile ch002_cert_extensions
