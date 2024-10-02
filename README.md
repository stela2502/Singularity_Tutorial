# Singularity/Apptainer Workshop Materials

The built documentation can be found here: https://singularity-tutorial.readthedocs.io/.

This repository contains the teaching materials for the **Singularity/Apptainer course**, hosted by the Lund Bioinformatics Core Unit in 2024. The course is designed to introduce participants to the process of building, managing, and deploying software using Apptainer (formerly Singularity) containers, with a focus on HPC environments and deployment strategies on the **COSMOS-SENS platform**.

## Overview

The workshop is aimed at researchers, bioinformaticians, and IT professionals who need to package and deploy software in a reproducible, self-contained environment. Throughout the course, participants will learn how to create and manage Apptainer images in unprivileged environments, typically required for high-performance computing (HPC) systems, and develop a deployment strategy for the COSMOS-SENS platform.

### Course Topics

1. **Introduction to Apptainer and Containerization**  
   - Understanding containers and their importance in reproducibility.
   - Why Apptainer? Differences between Apptainer and Docker.
   
2. **Building Apptainer Images**  
   - General overview of the Apptainer build process.
   - Crafting a `Singularity.def` file.
   
3. **Unprivileged Image Building in HPC Environments**  
   - Challenges of building containers in HPC systems.
   - Techniques for building containers without root access.
   
5. **Deploying Apptainer Containers on the COSMOS-SENS Platform**  
   - Integrating Apptainer with COSMOS-SENS.
   - Overview of deployment strategies for high-throughput analysis.
   - Managing image versions and updates in production environments.
   
6. **Hands-on Exercises and Real-world Use Cases**  
   - Practical examples for building and deploying bioinformatics pipelines using Apptainer.

## Setup Instructions

To follow along with the course materials and exercises, please ensure you have the following installed:

- **Open COSMOS access**: The course will use open COSMOS to build the images. 
- **COSMOS-SENS access**: For deployment exercises, contact Shamit Soneji if you not already have access; at least three weeks prior to the course.

## Repository Structure

- `docs/`: MkDocs-based documentation for the course material.
  
## Building and Serving MkDocs HTML Pages

The workshop materials are managed using [MkDocs](https://www.mkdocs.org/), a static site generator that's ideal for creating project documentation.

### Prerequisites

Ensure you have **Python** and **MkDocs** installed on your system:

```
pip install mkdocs
```

Optionally, you can also install the MkDocs Material theme, which offers a more modern look and feel:

```
pip install mkdocs-material
```

### Building the HTML Documentation

To generate the HTML pages from the markdown files in the `docs/` directory:

1. Navigate to the repository's root directory.
2. Run the following command to build the static site:

```
mkdocs build
```

This will create a `site/` directory containing the HTML files. You can deploy these HTML files to any web server or GitHub Pages.

### Serving the MkDocs Site Locally

To preview the site locally while you work on the documentation:

```
mkdocs serve
```

This will start a local development server, and you can view the site in your browser at `http://127.0.0.1:8000/`. Any changes made to the markdown files will automatically refresh the site.

### Deploying to GitHub Pages

If you're hosting the documentation on GitHub Pages, you can easily deploy it with MkDocs:

```
mkdocs gh-deploy
```

This command will build the documentation and push it to the `gh-pages` branch of your GitHub repository.

## Contribution Guidelines

We welcome contributions and improvements to the course materials. Please submit any issues or pull requests via the repository.

### License

This material is licensed under the [MIT License](LICENSE).

---

<!--
For further inquiries or support, please contact the **Lund Bioinformatics Core Unit** at [email@example.com].
-->

