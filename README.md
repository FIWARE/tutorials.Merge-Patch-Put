[![FIWARE Banner](https://fiware.github.io/tutorials.Merge-Patch-Put/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/FIWARE/tutorials.Getting-Started.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)

This tutorial introduces the NGSI-LD Merge Patch endpoint. It explains the difference between Merge Patch
(`/entities/<id>`) and Partial Update Patch (`/entities/<id>/attrs`) and demonstrates the use of this functionality.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://www.postman.com/downloads/)

# Start-Up

## NGSI-LD Smart Farm

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this tutorial with **NGSI-LD**, use the `NGSI-LD` branch.

```console
git clone https://github.com/FIWARE/tutorials.Merge-Patch-Put.git
cd tutorials.Merge-Patch-Put
git checkout NGSI-LD

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) | :books: [Documentation](https://github.com/FIWARE/tutorials.Merge-Patch-Put/tree/NGSI-LD) | <img  src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Merge-Patch-Put/ngsi-ld.html) |
| --- | --- | --- |


---

## License

[MIT](LICENSE) © 2022-2023 FIWARE Foundation e.V.
