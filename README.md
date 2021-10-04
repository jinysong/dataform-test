# Project Introduction

This is a simple Dataform exercise turning 2021 Olympics data on Kaggle into reporting tables in a BigQuery data warehouse.

## Summary

This project transforms 3 raw datasets (Athletes, Coaches, and Medals) into one reporting table.

- Country_Discipline combines the number of athletes and coaches for each country and discipline.
- Medals_country combines Country_Discipline with the number of medals won by each country.

## Dataset

https://www.kaggle.com/azmainmorshed/2021-olympics-analysis/data?select=Athletes.xlsx

## Dependency Tree

![image](https://user-images.githubusercontent.com/19098787/135849420-cf087d9c-da82-4f89-87fd-1c3be9755bf6.png)

## Directory Setup

![image](https://user-images.githubusercontent.com/19098787/135850179-fadc3f1c-6950-4f9e-9956-3c247097c36e.png)

## Final Reporting Table in BigQuery

![image](https://user-images.githubusercontent.com/19098787/135850374-cb72549d-7b6a-4020-90c0-c9a796b6284c.png)

# [Setup using CLI](https://docs.dataform.co/dataform-cli)

## create project
1. in github: create empty repo
2. in GCP: create new project
3. create new project using query ```dataform init bigquery new_project --default-database <your-google-cloud-project-id>```
```
# project directory

project-dir
├── definitions
├── includes
├── package.json
└── dataform.json
```
4. set git remote

### First time setups
- install Dataform CLI with ```npm i -g @dataform/cli```
- [Installing Cloud SDK (only once)](https://cloud.google.com/sdk/docs/install)
  - unpack .tar.gz ```tar xopf google-cloud-sdk-359.0.0-darwin-x86_64.tar.gz```
  - must move directories into root folder```mv hello/ ~/```
  - restart terminal to use gcloud

### connecting project to bigquery using application default
- there are two ways of connecting to bigquery: through application-default or through a service account json key
- in project terminal: ```gcloud auth application-default login```
- connect dataform project to bigquery ```dataform init-creds bigquery```
- add .df-credentials.json to .gitignore

### check connection
```
echo "config { type: 'view' } SELECT 1 AS test" > definitions/example.sqlx

dataform compile

# dataform run --dry-run
dataform run
```
This create a new view called 'example' in the 'dataform' dataset in BigQuery


# Tutorial

## Declarations using external datasets

```
config {
  type: "declaration",
  database: "dataform-test-327822",
  schema: "raw",
  name: "Athletes",
  description: "athletes from the 2021 olmypics"
}
```

## Staging data by creating temporary views

```
config {
  type: "view",
  schema: "staging",
  description: "athletes by discipline"
}

SELECT
    count(string_field_0) as num_Athletes,
    string_field_1 as Country,
    string_field_2 as Discipline
FROM ${ref("Athletes")}
GROUP BY 2,3
```

## Compile Tables for Reporting

```
config {
  type: "table",
  schema: "reporting",
  description: "add the number of medals for each country",
}

select
  Team_NOC as Country,
  count(Discipline) as num_Disciplines,
  sum(num_Athletes) as num_Athletes,
  sum(num_Coaches) as num_Coaches,
  Rank,
  Gold,
  Silver,
  Bronze,
  Total

from ${ref("Country_Discipline")} as Country_Discipline
join ${ref("Medals")} as Medals
on Country_Discipline.Country = Medals.Team_NOC
group by 1,5,6,7,8,9
```

## Compile & Run
```
dataform help

dataform compile
dataform compile --json # To see the output of the compilation process as a JSON object
dataform compile --vars=exampleVar=exampleValue,foo=bar # pass custom compilation variables

dataform run --dry-run
dataform run
```

## Document with descriptions in config block

Descriptions can be added for both tables/views and individual columns.
```
config {
  type: "table",
  schema: "reporting",
  description: "add the number of medals for each country",
}
```

## Test with assertions in config block

- Add tests in the config block
- Built-in tests include: uniqueKey, nonNull, and rowConditions
- Manuals assertions using SQL

```
# tests

config {
  type: "table",
  schema: "reporting",
  assertions: {
    uniqueKey: ["Country"],
    nonNull: ["Country", "Rank"],
    rowConditions: [
      'Rank > 0',
      'Total like "[0-9]+"'
    ]
  }
  description: "add the number of medals for each country",
}
```

```
# Manuals assertions using SQL


config { type: "assertion" }

SELECT
  *
FROM
  ${ref("sometable")}
WHERE
  a IS NULL
  OR b IS NULL
  OR c IS NULL
```

## Reduce code redundancy with Javascript


## BigQuery considerations (partition & clustering)
