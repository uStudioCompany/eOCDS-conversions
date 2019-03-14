### Background

Under preparation of the contract, notice CA has to indicate an awarding methodology which will be used to award a winning offer. OCDS covers this part of the notice with *awardCriteria*. But this approach only indicates a methodology, without describing a set of used criteria. To allow CA to include such set into the Contract Notice, separate OCDS extension has to be used - *ocds_requirements_extension*

- Original repository for [ocds_requirements_extension](https://github.com/open-contracting-extensions/ocds_requirements_extension)

Initially, this extension covers so-called exclusion grounds and selection criteria or together - ESPD. Here each EO who wants to participate in this specific contracting process has to complete a ‘form’ by adding either boolean answer (yes/no) or fill some specific data (like yearly turnover, the quantity of employees etc). 

But in case of awarding methodology based on quality or rated criteria its not enough. Additionally, quantitative criteria and applicable options and weights have to be described as well. For this reason, a separate internal extension can be applied - *conversions*

# Conversions

As soon as not only quantitative criteria has to be included into the Contract Notice but also - applicable options and weights, a separate extension has to be applied to allow CA to include all the conversions needed for future qualification and evaluation. 

*Conversions* - is a tool that allows describing used conversions and its applicable coefficients.

- to describe used *conversions* and its applicable *coefficients* either as a list of precise values or as a mathematical formula for calculation of the value of particular *coefficient* in this particular case (depending on the value received within *requirementResponse* related to specific *requirement*) to be applied
- to relate each *conversion* used (together with *coefficients*) with used *criteria* or *targets* (where applicable)
- to include applicable *options* to used *criteria* or *observations* for *targets*

### True/false requirement and its coefficients of conversion

The simple criterion that requires only true/false answer can be used by CA. For example - if currently submitting EO is a domestic bidder, his offer will get a ratio that increases the advantage of its price by 20%: 

```
{
  "criteria": [
    {
      "id": "001",
      "title": "Benefits",
      "description": "Benefits domestic bidders",
      "source": "tenderer",
      "relatesTo": "tenderer",
      "requirementGroups": [
        {
          "id": "001-1",
          "requirements": [
            {
              "id": "001-1-1",
              "title": "Is Economic operator is domestic bidder?",
              "description": "",
              "dataType": "boolean"
            }
          ]
        }
      ]
    }
  ]
}
```

Using *ocds_requirements_extension* we can describe this criterion as such. But using *conversions* we can also describe applicable coefficients:

```
{
  "conversions": [
    {
      "relatesTo": "requirement",
      "relatedItem": "001-1-1",
      "rationale": "Domestic bidders receive a 20% price preference",
      "coefficients": [
        {
          "value": true,
          "coefficient": 0.8
        },
        {
          "value": false,
          "coefficient": 1
        }
      ]
    }
  ]
}
```

So if EO will respond that his company is a domestic bidder, using cross-reference through *requirement_id* we can extract an applicable coefficient - 0.8

### Requirement with a predefined set of coefficients of conversion for a specific criterion value

The criterion that requires a precise answer with detalization can be used by CA. For example - when minimum product warranty of 1 year is required for all bids but warranties of 2 years receive a 15% advantage and warranties of 3 years or more receive a 30% advantage: 

```
{
  "criteria": [
    {
      "id": "002",
      "title": "Product warranty",
      "description": "A minimum product warranty of 1 year is required for all bids. Warranties of 2 years receive a 15% advantage. Warranties of 3 years or more receive a 30% advantage.",
      "source": "tenderer",
      "relatesTo": "item",
      "relatedItem": "1",
      "requirementGroups": [
        {
          "id": "002-1",
          "requirements": [
            {
              "id": "002-1-1",
              "title": "A minimum product warranty of 1 year is guaranteed",
              "dataType": "boolean",
              "expectednValue": true
            },
            {
              "id": "002-1-2",
              "title": "The number of years for proposed product warranty",
              "dataType": "integer",
              "minValue": 1,
              "maxValue": 3
            }
          ]
        }
      ]
    }
  ]
}
```

Using *ocds_requirements_extension* we can describe this criterion as such where EO  is required to confirm that he guarantees at least 1 year of product warranty (002-1-1) but also specify a precise number of years for such guarantee for the proposed product (002-1-2). 

And using *conversions* we can also describe applicable coefficients:

```
{
  "conversions": [
    {
      "relatesTo": "requirement",
      "relatedItem": "002-1-2",
      "rationale": "Number of years for product guarantee",
      "description": "",
      "coefficients": [
        {
          "value": 1,
          "coefficient": 1
        },
        {
          "value": 2,
          "coefficient": 0.85
        },
        {
          "value": 3,
          "coefficient": 0.7
        }
      ]
    }
  ]
}
```

Depending on EOs response we will have an applicable coefficient for future conversion.

### Requirement with a formula-based (calculated) coefficients of conversion for a specific criterion value

For the same criterion described above (product warranty) using *conversions* we can also describe coefficients for conversion not like a predefined set of values but as a mathematical formula that will be used to calculate applicable value:

```
{
  "conversions": [
    {
      "relatesTo": "requirement",
      "relatedItem": "002-1-2",
      "rationale": "Number of years for product guarantee",
      "description": "",
      "coefficients": [
        {
          "formula": "1.15-(value/10)*15"
        }
      ]
    }
  ]
}
```

Using mentioned formula and having a value for option inputed by EO withing his bid we can calculate applicable value for the coefficient of conversion:

- for 1 year: 1.15 - (1/10) * 15 = 1
- for 2 years: 1.15 - (2/10) * 15 = 0.85
- for three years: 1.15 - (3/10) * 15 = 0.7

As you can see - the result is the same 

### Requirement with available options to choose

Additionally *procuriosity_conversions* covers a following: ability to include an applicable *options* into used requirement. Such an approach allows prescribing more precise ranges of expected values together with specific coefficients of conversion. 

For example: if CA requires EO to have at least one Google certified specialist on his board but ready to grant EO with additional advantage if such EO has more than one guy:

```
{
  "criteria": [
    {
      "id": "003",
      "title": "Capacity",
      "description": "Minimum qualification requirements",
      "source": "tenderer",
      "relatesTo": "tenderer",
      "requirementGroups": [
        {
          "id": "003-1",
          "requirements": [
            {
              "id": "003-1-1",
              "title": "At least one Google certified specialist on-board",
              "dataType": "boolean",
              "expectedValue": true
            },
            {
              "id": "003-1-2",
              "title": "Number of Google certified staff",
              "description": "",
              "dataType": "integer",
              "minValue": 1,
              "options": [
                {
                  "maxValue": 3,
                  "title": "Up to three specialist"
                },
                {
                  "maxValue": 5,
                  "title": "Up to five specialists"
                },
                {
                  "minValue": 6,
                  "title": "More han five specialists"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

And the conversion will be the following: 

```
{
  "conversions": [
    {
      "relatesTo": "requirement",
      "relatedItem": "003-1-2",
      "rationale": "Number of Google certified staff",
      "description": "",
      "coefficients": [
        {
          "minValue": 1,
          "coefficient": 1
        },
        {
          "minValue": 4,
          "coefficient": 0.99
        },
        {
          "minValue": 6,
          "coefficient": 0.98
        }
      ]
    }
  ]
}
```

## Coverage for targets

As shown in examples above, it is the following construction used instead of using *requirementReference* from *ocds_requirements_extension*. 

```
    "relatesTo": "",
    "relatedItem": "",
```

This is done for a purpose: in the same way, as it used for *requirements*, *conversions* could be used for another one attribute of contracting process that used to describe indicative or precise forecast of procurement - *targets*.

Using *relatesTo*-approach we can use both *options* and *conversions* for *targets* as well. 

For example, CA is going to buy 1000 cars during some period. Of course its much more convenient for him to buy it from 'one hand' but what if there is no one on the market how can deliver such quantity of cars? (yes - it goes to the lots, this is just an example :). In that case, CA can describe options, available for this target together with the conversion which will be applied for the lower offered quantity of cars:

```
{
  "targets": [
    {
      "id": "annualNeed",
      "title": "Annual need",
      "description": "The annual need",
      "observations": [
        {
          "period": {
            "startDate": "2015-01-01T00:00:00Z",
            "endDate": "2015-12-31T23:59:59Z"
          },
          "measure": 1000,
          "unit": {
            "id": "NC",
            "name": "car"
          },
          "options": [
            {
              "measure": "800"
            }
          ]
        }
      ]
    }
  ]
}
```

And the conversion will be:

```
{
  "conversions": [
    {
      "relatesTo": "target",
      "relatedItem": "annualNeed",
      "rationale": "Indexing of the increase in total cost of ownership",
      "description": "",
      "coefficients": [
        {
          "coefficient": 1.15
        }
      ]
    }
  ]
}
```

So we can see that EO who offers less than 1000 cars will be accepted for evaluation but with 15% lower offer advantage (just due to the increasing total cost of ownership for CA).

A similar approach could be applied where CA needs to tie his spendings with the achieved level of  KPIs by supplier while implementing the contract (e.g. energy efficiency services) or to achieve the best value for money under 'quality-only' procedures.
