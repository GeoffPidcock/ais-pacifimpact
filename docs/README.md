# Table of Contents

- [Getting Started](#Setup)
- [Data](#Data)

# Setup
1. Clone source code

`git clone https://github.com/GeoffPidcock/ais-pacifimpact`

2. Setup environment

**Python**
Create python=3.8
```
conda create -- name pacifimpact python=3.8
conda activate pacifimpact
```
- install python requirements
`pip install -r requirements.txt`


**R**
- [install R](https://www.r-project.org/)
   - This code uses `R version 3.6.3 (Holding the Windsock)`
- install and setup [R studio](https://rstudio.com/)
- open `ais-pacifimpact.Rproj`


3. Request secrets for database access

Ask the team to share 
- `.env` file containing secrets for Python, and 
- `.Renviron` containing secrets for R


# Data
Download Data from google drive and move to `./data/raw` project directory
[The data is fully attributed in this reference](https://docs.google.com/spreadsheets/d/1qKgOixdJtwYD0jB1ouN4-pUIqrEXEPJzAqWfYbIY06g/edit?usp=sharing)
[Link (request access)](https://drive.google.com/drive/folders/182s8YXv7o9Unnd3Z69APOxA0hiCWqPDn?usp=sharing)