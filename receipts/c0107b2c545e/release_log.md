# Release log: proteomics v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:53:31Z | 2f08ddb68a93d56e563468e1ad95898192057fd9 | `{"candidate_tree": "sha256:1087214466b9181b3802cff81186ace612ed0b51c11effe40ea15e046f5d6e6d", "gate_report": "1e513ff272debcdf52e5504a4f1ee41d1bd643c90013bfa959a69521624ed6b8"}` |
| deposit-plan | 2026-11-09T14:53:31Z | 2f08ddb68a93d56e563468e1ad95898192057fd9 | `{"archive_sha256": "fb55d65edee9ece4f945425166ebf382f2f64091d4fb30869da50091fc0ff1a6"}` |
| deposit-draft | 2026-11-09T21:06:07Z | 2f08ddb68a93d56e563468e1ad95898192057fd9 | `{"archive_sha256": "fb55d65edee9ece4f945425166ebf382f2f64091d4fb30869da50091fc0ff1a6"}` |
| record-doi | 2026-11-10T08:01:09Z | 2f08ddb68a93d56e563468e1ad95898192057fd9 | `{"CITATION.cff": "bf55e064a4c5dbce6f6243a7a72e961ec373901fc755e4c666581ba01c5300bd", "README.md": "a53f5c304ecde41e193d7907bcf6c208555fd138750ba6e69de1cc110d90e2a0", "RELEASE.yaml": "d61260ceca0bf283100c83076af6ae477c71be9b6f57b74910728916df389da7", "marketplace.json": "8a07b750b3cb4dfa4a1710b47817b80d81423bc21396f0a1e4df349e546c3e49", "plugin.json": "44653d4f84916e893e75378689fee0cca61e66ddbde3d2957c7b79d65983b169"}` |
| receipt | 2026-11-10T08:01:41Z | 2f08ddb68a93d56e563468e1ad95898192057fd9 | `{"receipt": "46f233a8e4ca7d920b580765e3bcd375434cc87ab63a12c6f93a4bbacb9295fd"}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093114; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
