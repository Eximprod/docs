# ES200 documentation <!-- omit from toc -->

- [1. Overview](#1-overview)
- [2. Quick links](#2-quick-links)
- [3. Extensions](#3-extensions)
  - [3.1. Markdown All in One](#31-markdown-all-in-one)
    - [3.1.1. Overview](#311-overview)
    - [3.1.2. Links](#312-links)
    - [3.1.3. Motivation](#313-motivation)
    - [3.1.4. Installation](#314-installation)
    - [3.1.5. Usage](#315-usage)
  - [3.2. Save and Run](#32-save-and-run)
    - [3.2.1. Overview](#321-overview)
    - [3.2.2. Links](#322-links)
    - [3.2.3. Motivation](#323-motivation)
    - [3.2.4. Installation](#324-installation)


## 1. Overview

ES200 is a control, monitoring and data acquisition solution for the automation of field equipment and processes.

Using modern and secure communication and automation standards, the ES200 is designed to efficiently operate power grids, oil and gas networks, manufacturing units, smart city solutions and so on.

To demonstrate the ES200 solution, a Proof of Concept can be deployed to evaluate the solution's fit to the business and technology requirements. This document explains the PoC process and its requirements.

## 2. Quick links

- [ES200 Dashboard Manual EN](ES200/ES200_Dashboard_Manual.md)
- [ES200 Dashboard Manual RO](ES200/ES200_Dashboard_Manual_RO.md)
- [ES200 Installation Manual](ES200/ES200_Installation_Manual.md)
- [ES200 REST API Manual](ES200/ES200_REST_API_Manual.md)
- [HMI Manual](HMI/HMI_Manual.md)
- [Proof Of Concept](PoC/Proof_Of_Concept.md)


## 3. Extensions

### 3.1. Markdown All in One

Markdown All in One is a Visual Studio Code extension that improves Markdown editing by offering real-time previews, automatic table of contents, linting for errors, and handy shortcuts. It streamlines the Markdown workflow and can be customized to fit your needs.

#### 3.1.1. Overview
This extension is mandatory. It is used to automatically update the section numbers and generate/update the Table of Contents.

#### 3.1.2. Links
- [VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
- [GitHub repository](https://github.com/yzhang-gh/vscode-markdown?tab=readme-ov-file#latest-development-build)


#### 3.1.3. Motivation
Remove manual work and possible human errors by not writing every Chapter's number and update the Table of Contents whenever a Markdown document is updated (manually).

#### 3.1.4. Installation
This extension is automatically installed by a VSCode task (OS independent).

#### 3.1.5. Usage
The user will have to write the Chapters using **#**. Each subsequent use of multiple **#** will define a subchapter of the first one.

The user will use the shortcut **CTRL + Shift + P** and select the appropriate option:
1. **Add/Update Section Numbers**: automatically update the sections numbers for each Chapter/Subchapter. 
2. **Create Table of Contents**: automatically creates the ToC. If this already exists, use option **2**, to update the existing ToC.
3. **Update Table of Contents**: automatically updates the existing ToC.

Both **Section Numbers** and **Table of Contents** will also update themselves whenever the document is saved **CTRL + S**. This can be disabled via TODO.

### 3.2. Save and Run
TODO

#### 3.2.1. Overview
TODO

#### 3.2.2. Links
- [VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=padjon.save-and-run-ext)
- [GitHub repository](https://github.com/padjon/vscode-save-and-run-ext)

#### 3.2.3. Motivation
TODO

#### 3.2.4. Installation
TODO
