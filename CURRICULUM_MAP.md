# Curriculum Map

## Raw Repository Structure

```bash
── .gitignore
├── docs
│   ├── .DS_Store
│   ├── index.md
│   ├── setup
│   │   ├── .DS_Store
│   │   ├── images
│   │   │   ├── orbstack_newmachine_menu.png
│   │   │   ├── vsc_explorer_openFolder.png
│   │   │   ├── vsc_extension_dockerInstall.png
│   │   │   ├── vsc_extension_remoteExplorer.png
│   │   │   └── vsc_remoteExplorer_sshConnect.png
│   │   └── su-readme.md
│   └── workflow
│       ├── .DS_Store
│       ├── images
│       │   ├── avd-workflow-diagram.png
│       │   ├── avd_cv_deploy_diagram.png
│       │   ├── avd_eos_cli_config_gen_diagram.png
│       │   ├── avd_eos_designs_role_diagram.png
│       │   ├── cvaas_cc_approve.png
│       │   ├── cvaas_cc_pending.png
│       │   ├── cvaas_cc_successful.png
│       │   └── cvaas_studio_workspace_submitted.png
│       └── wf-readme.md
└── src
    ├── .DS_Store
    ├── clab_avd_campus_topo.yml
    ├── cvp_extensions
    │   └── TerminAttr64-1.36.1-1.swix
    └── startup_config
        └── startup-config
```

## Module 1.1 – Lesson Mapping

| Repo Path                                               | File Type  | Maps to Lesson                         | Role       | Status  | Notes                                                      |
| ------------------------------------------------------- | ---------- | -------------------------------------- | ---------- | ------- | ---------------------------------------------------------- |
| `.gitignore`                                            | config     | —                                      | internal   | usable  | No curriculum impact                                       |
| `docs/index.md`                                         | markdown   | 1.1.1 Lab Architecture Overview        | primary    | partial | Good entry point, needs clearer “what this lab is / isn’t” |
| `docs/setup/su-readme.md`                               | markdown   | 1.1.2 Host Preparation & Validation    | primary    | usable  | Strong base, needs pre-flight checklist                    |
| `docs/setup/images/*`                                   | images     | 1.1.2 Host Preparation                 | supporting | usable  | Great for beginners; optional in paid tier                 |
| `docs/workflow/wf-readme.md`                            | markdown   | 1.1.1 / 1.1.6                          | supporting | partial | More “AVD workflow” than lab setup                         |
| `docs/workflow/images/avd-workflow-diagram.png`         | image      | 1.1.1                                  | supporting | usable  | Excellent conceptual asset                                 |
| `docs/workflow/images/avd_eos_designs_role_diagram.png` | image      | 1.1.1                                  | supporting | usable  | Strong tie-in to campus roles                              |
| `docs/workflow/images/avd_cv_deploy_diagram.png`        | image      | 1.1.6                                  | supporting | usable  | Slightly ahead of Module 1.1 scope                         |
| `docs/workflow/images/cvaas_*`                          | images     | 1.1.6                                  | reference  | usable  | Likely Tier-1+ content                                     |
| `src/clab_avd_campus_topo.yml`                          | yaml       | 1.1.4 Containerlab Topology Definition | primary    | usable  | Critical artifact — needs annotation                       |
| `src/startup_config/startup-config`                     | eos config | 1.1.6 Initial Lab Bring-Up             | supporting | usable  | Clarify why it exists                                      |
| `src/cvp_extensions/TerminAttr64-1.36.1-1.swix`         | binary     | 1.1.5 cEOS Image Handling              | reference  | usable  | Needs explanation + licensing context                      |

## Identified Gaps for Module 1.1

### Required but Missing

- Host pre-flight checklist
- Supported OS/version table
- Failure taxonomy
- Reset / teardown procedure

### Exists but Needs Expansion

- Topology annotation
- Image handling guidance

## Lesson Coverage Heat Map

This shows what’s strong vs weak in Module 1.1.

| Lesson                          | Coverage   | Notes                                      |
| ------------------------------- | ---------- | ------------------------------------------ |
| 1.1.1 Lab Architecture Overview | 🟡 Partial | Concepts exist, narrative needs tightening |
| 1.1.2 Host Prep & Validation    | 🟢 Strong  | One of your strongest sections             |
| 1.1.3 Repo & File Structure     | 🔴 Missing | Needs a new short doc                      |
| 1.1.4 Topology Definition       | 🟢 Strong  | YAML exists, needs teaching layer          |
| 1.1.5 cEOS Image Handling       | 🟡 Partial | Image present, explanation missing         |
| 1.1.6 Initial Bring-Up          | 🟡 Partial | Workflow docs bleed into later modules     |
| 1.1.7 Troubleshooting           | 🔴 Missing | High-value gap                             |
| 1.1.8 Reset & Reuse             | 🔴 Missing | Easy win                                   |

## Presentation Staging Assessment (Module 1.1)

This helps you decide what goes public first.

| Lesson                | Public Now | Needs Cleanup | Internal Only |
| --------------------- | ---------- | ------------- | ------------- |
| 1.1.1 Architecture    | ⬜         | ☑️            | ⬜            |
| 1.1.2 Host Prep       | ☑️         | ⬜            | ⬜            |
| 1.1.3 Repo Structure  | ⬜         | ☑️            | ⬜            |
| 1.1.4 Topology        | ⬜         | ☑️            | ⬜            |
| 1.1.5 Image Handling  | ⬜         | ☑️            | ⬜            |
| 1.1.6 Bring-Up        | ⬜         | ☑️            | ⬜            |
| 1.1.7 Troubleshooting | ⬜         | ⬜            | ☑️            |
| 1.1.8 Reset & Reuse   | ⬜         | ⬜            | ☑️            |
