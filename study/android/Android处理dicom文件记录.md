---
title: Android处理dicom文件记录
date: 2021-10-29 17:30:55
categories:
    - Android
tags:
    - Android
    - dicom
    - dcm
---

# dcm4che

纯java编写，网上有搜到android版本的jar包，是dcm4che低版本，只在安卓平台上实现了reader,没实现writer。github上的版本有reader和writer实现，但是是依赖javax相关包的，安卓上没有这些包。所以用不了github上的版本。



## Modules

- dcm4che-audit
- dcm4che-audit-keycloak
- dcm4che-conf
  - dcm4che-conf-api
  - dcm4che-conf-api-hl7
  - dcm4che-conf-json
  - dcm4che-conf-json-schema
  - dcm4che-conf-ldap
  - dcm4che-conf-ldap-audit
  - dcm4che-conf-ldap-hl7
  - dcm4che-conf-ldap-imageio
  - dcm4che-conf-ldap-schema
- dcm4che-core
- dcm4che-dcmr
- dcm4che-deident
- dcm4che-dict
- dcm4che-dict-priv
- dcm4che-emf
- dcm4che-hl7
- dcm4che-image
- dcm4che-imageio
- dcm4che-imageio-opencv
- dcm4che-imageio-rle
- dcm4che-js-dict
- dcm4che-json
- dcm4che-mime
- dcm4che-net
- dcm4che-net-audit
- dcm4che-net-hl7
- dcm4che-net-imageio
- dcm4che-soundex
- dcm4che-ws-rs
- dcm4che-xdsi
- dcm4che-jboss-modules

## Utilities

- [agfa2dcm](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-agfa2dcm/README.md): Extract DICOM files from Agfa BLOB file

- [dcm2dcm](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcm2dcm/README.md): Transcode DICOM file according the specified Transfer Syntax

- [dcm2jpg](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcm2jpg/README.md): Convert DICOM image to JPEG or other image formats

- [dcm2json](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcm2json/README.md): Convert DICOM file in JSON presentation

- [dcm2pdf](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcm2pdf/README.md): Extract encapsulated PDF, CDA or STL from DICOM file

- [dcm2str](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcm2str/README.md): Apply Attributes Format Pattern to dicom file or command line parameters.

- [dcm2xml](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcm2xml/README.md): Convert DICOM file in XML presentation

- [dcmdir](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcmdir/README.md): Dump, create or update DICOMDIR file

- [dcmdump](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcmdump/README.md): Dump DICOM file in textual form

- [dcmldap](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcmldap/README.md): Insert/remove configuration entries for Network AEs into/from LDAP server

- [dcmqrscp](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcmqrscp/README.md): Simple DICOM archive

- [dcmvalidate](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-dcmvalidate/README.md): Validate DICOM object according a specified Information Object Definition

- [deidentify](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-deidentify/README.md): De-identify one or several DICOM files

- [emf2sf](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-emf2sf/README.md): Convert DICOM Enhanced Multi-frame image to legacy DICOM Single-frame images

- [findscu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-findscu/README.md): Invoke DICOM C-FIND Query Request

- [getscu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-getscu/README.md): Invoke DICOM C-GET Retrieve Request

- [hl72xml](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-hl72xml/README.md): Convert HL7 v2.x message in XML presentation

- [hl7pdq](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-hl7pdq/README.md): Query HL7 v2.x Patient Demographics Supplier

- [hl7pix](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-hl7pix/README.md): Query HL7 v2.x PIX Manager

- [hl7rcv](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-hl7rcv/README.md): HL7 v2.x Receiver

- [hl7snd](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-hl7snd/README.md): Send HL7 v2.x message

- [ianscp](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-ianscp/README.md): DICOM Instance Availability Notification receiver

- [ianscu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-ianscu/README.md): Send DICOM Instance Availability Notification

- [jpg2dcm](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-jpg2dcm/README.md): Convert JPEG images or MPEG videos in DICOM files

- [json2props](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-json2props/README.md): Convert Archive configuration schema JSON files to key/value properties files and vice versa

- [json2rst](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-json2rst/README.md): Generate ReStructuredText files from Archive configuration schema JSON files

- [mkkos](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-mkkos/README.md): Make DICOM Key Object Selection Document

- [modality](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-ihe/dcm4che-tool-ihe-modality/README.md): Simulates DICOM Modality

- [movescu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-movescu/README.md): Invoke DICOM C-MOVE Retrieve request

- [mppsscp](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-mppsscp/README.md): DICOM Modality Performed Procedure Step Receiver

- [mppsscu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-mppsscu/README.md): Send DICOM Modality Performed Procedure Step

- [pdf2dcm](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-pdf2dcm/README.md): Convert PDF file into DICOM file

- [stgcmtscu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-stgcmtscu/README.md): Invoke DICOM Storage Commitment Request

- [storescp](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-storescp/README.md): DICOM Composite Object Receiver

- [storescu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-storescu/README.md): Send DICOM Composite Objects

- [stowrs](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-stowrs/README.md): Send DICOM Composite Objects or Bulkdata file over Web

- [stowrsd](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-stowrsd/README.md): STOW-RS Server

- [swappxdata](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-swappxdata/README.md): Swaps bytes of uncompressed pixel data in DICOM files

- [syslog](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-syslog/README.md): Send Syslog messages via TCP/TLS or UDP to a Syslog Receiver

- [syslogd](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-syslogd/README.md): Receives RFC 5424 Syslog messages via TCP/TLS or UDP

- [upsscu](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-upsscu/README.md): Invokes services of Unified Procedure Step Service Class

- [wadors](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-wadors/README.md): Wado RS Client Simulator

- [wadows](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-wadows/README.md): Wado WS Client Simulator

- [xml2dcm](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-xml2dcm/README.md): Create/Update DICOM file from/with XML presentation

- [xml2hl7](https://github.com/dcm4che/dcm4che/blob/master/dcm4che-tool/dcm4che-tool-xml2hl7/README.md): Create HL7 v2.x message from XML presentation

  

license: Mozilla Public License Version 1.1

# imebra

Imebra now supplies the pre-compiled library for Android, which applies the Windowing in real time when you use the DicomView class (an Android View that displays the Dicom Image).

There is no need to convert to jpeg: Follow the example in the documentation and then modify the windowing value in the voilut object.

license:To use Imebra without being bound to the **GPLv2** license you have to **buy a commercial license**. https://imebra.com/buy/

# [PixelMed](http://www.pixelmed.com/)

stackoverflow上有帖子说可以用在安卓上，相关资料很少

# DCMTK

论坛说可以用ndk编译，跑在安卓上 

 Compiling DCMTK for Android(中文版)
 https://blog.csdn.net/chenhuakang/article/details/73608510

license:A free evaluation licence is available for the OFFIS DICOM software packages DCMPPS, DCMPPSCU, PPSMGR, DCMPRINT SCU, DCMPRINT SCP, DCM2AVI, DCMJP2K and DCMSTCOM. The evaluation licence allows to download the software from the internet and to evaluate it for a period of four months. Any further use of the software requires a full licence agreement.