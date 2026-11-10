# Release log: epigenomics v2

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:56:08Z | ae9feed1efbe760d5d4afda0bde54eac177b80b2 | `{"candidate_tree": "sha256:10b457f5276ded5de273e1132842ac6ecedbd6f5756bac6bd1b2c2da4d50c472", "gate_report": "0ba1136af2fea47554ef1b37f17fb69446cce1977a49ae14bd7cc8425b986b9a"}` |
| deposit-plan | 2026-11-09T14:56:09Z | ae9feed1efbe760d5d4afda0bde54eac177b80b2 | `{"archive_sha256": "def189e660101d0aac7cb9256b4802bc7735d082d740372c3e5119476d4ee5fc"}` |
| deposit-draft | 2026-11-09T21:06:03Z | ae9feed1efbe760d5d4afda0bde54eac177b80b2 | `{"archive_sha256": "def189e660101d0aac7cb9256b4802bc7735d082d740372c3e5119476d4ee5fc"}` |
| record-doi | 2026-11-10T10:48:44Z | ae9feed1efbe760d5d4afda0bde54eac177b80b2 | `{"CITATION.cff": "b9581d28d9b74c5ba7ee2e71eb6681bd59dd457157934341af53325b73cfe88c", "README.md": "f049bd7de1da8956dd0fa3b313ba56c5e4bcb51e5bf84b8d697ce0aafa495e24", "RELEASE.yaml": "59ffc807b96ef09ee6be2d74ef93146f86c3890efcadb95fc1ba4be5f98a1551", "marketplace.json": "02bb3ffb9c597848ead4d5f638ed428693f7c276f1c49f541721e4cfd15fbb43", "plugin.json": "7aeec0fdf56801398c2d138ccf33cd33a92138b0d50dfbc43e0ba6dd9bf7f513"}` |
| receipt | 2026-11-10T10:50:08Z | ae9feed1efbe760d5d4afda0bde54eac177b80b2 | `{"receipt": "38c9d20ce876dd65ec1a081f63e61669affc936d8248d2cca9078d845a4c70e6"}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093107; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
