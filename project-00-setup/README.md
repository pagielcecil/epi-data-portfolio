# Project 00: Setup check (Quarto to PDF)

## Question
Does my reproducible-reporting workflow (R, Quarto, Git, GitHub) work end to end?

## Data
H7N9 influenza cases in China, 2013. These are real, published public data
(`fluH7N9_china_2013`) from the R package `outbreaks`. No restricted or
confidential data are used.

## How to re-run
From the root of the repository, run:

```
quarto render project-00-setup/report/setup_check.qmd
```

Requirements: R (4.1 or newer), Quarto, TinyTeX, and the R packages
`tidyverse` and `outbreaks`.

## What the report shows
The epidemic curve shows a rapid surge in H7N9 cases in early 2013, peaking
around late March to early April, followed by a gradual decline. A key
limitation is that missing or delayed onset dates can distort the shape of the
curve and make the exact timing of the peak harder to pin down.

## Files
- `report/setup_check.qmd`: report source
- `report/setup_check.pdf`: rendered report











