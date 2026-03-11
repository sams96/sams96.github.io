---
title: ''
url: /photo/{{ .File.ContentBaseName }}
date:
datePosted: {{ .Date | time.Format "2006-01-02" }}
image: {{ .File.ContentBaseName }}.avif
type: photo
alt: >-
  TODO alt text

metadata:
  camera: Hasselblad 501c
  lens: Zeiss Planar 80mm f/2.8 CB | Sonnar 150mm f/4 CF | Distagon 60mm f/3.5 CF
  film: Ilford HP5+ @ 800
  location:
    - TODO
    - TODO
---
