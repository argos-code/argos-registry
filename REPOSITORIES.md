# Argos — managed repositories

This file is the source of truth for the repositories argos watches. To
start watching a repository, open a pull request adding a row to the table
below; to stop watching one, remove its row. Set the **UI testing** column
to `yes` to enable UI flow testing for that repository.

Removing a row disables the repository (argos stops watching it) but keeps
its stored history — permanent deletion is a CLI-only operation
(`argos repo remove`).

Argos also mirrors changes made through its CLI into this file, so the table
always reflects the live state. Edit only the table between the markers;
the surrounding text is yours.

<!-- argos:repos -->
| Repository | UI testing |
|---|---|
| vocdoni/dvote-protobuf | no |
| vocdoni/explorer-ng | yes |
| vocdoni/integrator-sdk | no |
| vocdoni/saas-backend | no |
| vocdoni/vocdoni-app | yes |
| vocdoni/vocdoni-leads-crm | no |
| vocdoni/vocdoni-node | no |
<!-- /argos:repos -->
