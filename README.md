[![FIWARE Banner](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Historic-Context-NIFI.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)

This tutorial is an introduction to [FIWARE Draco](https://fiware-draco.readthedocs.io/en/latest/) - an alternative
generic enabler which is used to persist context data into third-party databases using
[Apache NIFI](https://nifi.apache.org) creating a historical view of the context. The in the same manner asthe
[previous tutorial](https://github.com/FIWARE/tutorials.Historic-Context-NIFI), activates the dummy IoT sensors
persists measurements from those sensors into a database for further analysis.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://www.postman.com/downloads/).

# Start-Up

## NGSI-v2 Smart Supermarket

**NGSI-v2** offers JSON based interoperability used in individual Smart Systems. To run this tutorial with **NGSI-v2**, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.Historic-Context-NIFI.git
cd tutorials.Historic-Context-NIFI
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/) | :books: [Documentation](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/tree/NGSI-LD) | <img src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Historic-Context-NIFI/) |
| --- | --- | --- |

<!--
## NGSI-LD Smart Farm

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this tutorial with **NGSI-LD**, use the `NGSI-LD` branch.

```console
git clone https://github.com/FIWARE/tutorials.Historic-Context-NIFI.git
cd tutorials.Historic-Context-NIFI
git checkout NGSI-LD

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) | :books: [Documentation](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/tree/NGSI-LD) | <img  src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Historic-Context-NIFI/ngsi-ld.html) |
| --- | --- | --- |
-->

---

## License

[MIT](LICENSE) © 2018-2024 FIWARE Foundation e.V.
