# Singularity Tutorial / Image Builder

This repository provides a **simple, reproducible workflow** to create and manage
Singularity/Apptainer images using a **Makefile-based approach**.

It also contains a small helper script to **bootstrap new image projects** with
a consistent structure.

---

## 📖 Documentation

The full documentation how to use this can be found here:

👉 https://stela2502.github.io/Singularity_Tutorial/AMakefileBasedApproach.html

For a more general introduction into apptainer and it's usage on HPC systems:

👉 https://stela2502.github.io/Singularity_Tutorial/index.html


---

## 🚀 Quick Start

### 1. Create a new image project

The main entry point of this repository is:

```bash
./create_new_image_builder.sh <my_new_image>
```

Example:

```bash
./create_new_image_builder.sh my_tool
```

This will:

- Create a new directory `my_tool/`
- Copy template files into it
- Rename everything from `Bioinformatics` → `my_tool`

---

### 2. Generated structure

```
my_tool/
├── my_tool.def        # Singularity definition file (edit this!)
├── Makefile          # Build + run helpers
├── run.sh            # Execute container
├── shell.sh          # Interactive shell
├── generate_module.sh# Optional module file generator
└── .gitignore
```

---

### 3. Build the container

```bash
cd my_tool
make
```

---

### 4. Run it

```bash
./run.sh
```

Or open a shell:

```bash
./shell.sh
```

---

## 🧠 Design Philosophy

This repo avoids complex frameworks and instead uses:

- **Plain Makefiles** → transparent, debuggable
- **Singularity definition files (`.def`)** → reproducible builds
- **Minimal scripting** → no Python dependency mess

The goal is:

> Fast, reproducible, HPC-friendly container builds without magic.

---

## 🛠 How the Generator Works

The script:

```bash
create_new_image_builder.sh <name>
```

- Copies template files from `image/`
- Renames all occurrences of `Bioinformatics` → `<name>`
- Creates a ready-to-build project skeleton

You are expected to:

1. Edit the generated `.def` file
2. Adjust the `Makefile` and there especially DEPLOY_DIR and MODULE_FILE

That’s it.

---

## 📦 Requirements

- Singularity or Apptainer
- Bash
- Make

---

## 🔄 Documentation Build (local)

```bash
mdbook build -d site
mdbook serve
```

---

## 🤝 Notes

- Designed for **HPC environments (SLURM, modules, etc.)**
- Works well with **Apptainer on clusters**
- Keeps everything **explicit and versionable**

---

## 📜 License

MIT (or your preferred license)
