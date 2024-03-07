# Introduction
This project provides two APIs for an AD-serving system, including an admin API for managing AD resources and a public API for AD delivery.

*AD is an abbreviation of "advertisement"
# Configuration & Dependencies
# Usage Contract
## API

<table>
<tr>
<td> Description </td> <td> Endpoint </td> <td> Query Parameters </td> <td> Request Body </td> <td> Response Code </td>
<td> Response Body </td> <td>Expected Behavior</td> <td> Error Handling </td><td> Performance </td>
</tr>
<tr>
<td> Create an AD resource </td>
<td>

`PUT /api/v1/ad`

</td>
<td>N/A</td>
<td>

```json
{
    // Next section provide more details to each fields
    "title": "AD 55",
    "startAt": "2023-12-10T03:00:00.000Z",
    "endAt": "2023-12-31T16:00:00.000Z",
    "conditions": {
        "ageStart": 20,
        "ageEnd": 30,
        "gender": ["M", "F"],
        "country": ["TW", "JP"],
        "platform": ["android", "ios", "web"]
    }
}
```

</td>
<td> 201: The resource is created successfully <br> 400: Invalid request argument(s) <br> 422: Invalid argument value(s) </td>
<td>

```json
{
    "id": 1
}
```

</td>
<td> No duplicated checks. Users must check on his/her own cost. Useful APIs (getAdByName, getAdByKeywords) will be provided in the near future to assist users to check if the AD already exists. </td>
<td>
This API return status code 4XX or 5XX alongs with an error message if anything goes wrong.
</td>
<td>
It is not designed for batch requests (allowing user to add multiple ADs by one call) since there are no use cases that needs to add lots of ADs at a time and allowing batched requests complicate how users use the API.
</td>
</tr>
<tr>
<td>

Get all ADs that match specific criteria. Returned ADs are sorted in ascending order of `endAt`

</td>
<td>

`GET /api/v1/ad`

</td>
<td>

Optional filtering conditions include `offset`, `limit`, `age`, `gender`, `country`, `platform`.

</td>
<td>N/A</td>
<td> 200: Retrieve the ADs successfully <br> 400: Unrecognized condition(s) <br> 422: Invalid query value </td>
<td>

```json
{
    "ads": [
        {
            "title": "AD 1",
            "startAt": "2023-12-10T03:00:00.000Z",
            "endAt": "2023-12-31T16:00:00.000Z"
        }
    ],
    "hasNext": true,
    "nextOffset": 10
}
```

</td>
<td>

This API will not perform ring retrieval. For example, getting ADs from `offset` INT_MAX will not get ADs back from `offset` 0. The API will do its best to fulfill the `limit` unless `hasNext` equals to false. If `hasNext` equals to false, `nextOffset` will be `INT_MAX` to prevent clients accidentally get into an infinite loop.

</td>
<td>

This API return status code 4XX or 5XX alongs with an error message if anything goes wrong.

</td>
<td>
It guarentees to support 10,000 RPS.
</td>
</tr>
</table>

## Body/Query Arguments
* `PUT /api/v1/ad`
  * Request Body Arguments <br>
    **Required**
    | Argument | Meaning                          | Type   | Note                                                                                                |
    | -------- | -------------------------------- | ------ | --------------------------------------------------------------------------------------------------- |
    | title    | A brief description of the AD    | string | Need not to be unique                                                                               |
    | startAt  | The time to start showing the AD | string | The time format should comply with ISO 8601 standard in UTC time and should be earlier than `endAt` |
    | endAt    | The time to stop showing the AD  | string | The time format should comply with ISO 8601 standard in UTC time and should be later than `startAt` |

    **Optional**
    | Argument            | Meaning                                     | Type             | Note                                                                                                                                                        |
    | ------------------- | ------------------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | conditions          | When the conditions are met, ADs show       | object           | Conditions arguments are all optional, and omitting arguments mean no constaints                                                                            |
    | conditions.ageStart | The minimum age to target                   | integer          | Negative number or not providing this constraint is equivalent to providing 0. Need to be smaller than or equal to `conditions.ageEnd`                      |
    | conditions.ageEnd   | The maximum age to target                   | integer          | Negative number or not providing this constraint is equivalent to providing 100. Need to be larger than or equal to `conditions.ageStart`                   |
    | conditions.gender   | The legal gender of the target audience     | array of strings | Includes "M" for male, "F" for female; other gender identifiers may be supported in the future but are not allowed currently. Genders are case-insensitive. |
    | conditions.country  | The countries of the target audience        | array of strings | Uses ISO 3166-1 country codes. Countries are case-insensitive.                                                                                              |
    | conditions.platform | The platforms through which the AD is shown | array of strings | Only support "android", "ios", and "web" now. Platforms are case-insensitive.                                                                               |

* `GET /api/v1/ad`
  * Request query arguments <br>
    **Required**
    | Argument | Meaning                                       | Type    | Note                                                                                                                                                                                             |
    | -------- | --------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | offset   | The starting point from which to list the ads | integer | Specifies the starting index from which ads are listed. The AD with `offset` is included in the returned AD list. If `offset` is negative or larger than INT_MAX, error status code is returned. |
    | limit    | The maximum number of ads to list             | integer | Specifies the max number of ads to return in one response. Default 5, range 1-100. If it is larger than 100, it will be adjusted to 100. If it is smaller than 1, it will be adjusted to 1.      |

    **Optional**
    | Argument | Meaning                                    | Type    | Note                                                                            |
    | -------- | ------------------------------------------ | ------- | ------------------------------------------------------------------------------- |
    | age      | The age of the target audience             | integer | Used to filter ads targeting specific age groups                                |
    | gender   | The legal gender of the target audience    | string  | Can be "M" for male or "F" for female; filters ads by gender. Case-insensitive. |
    | country  | The country of the target audience         | string  | Uses ISO 3166-1 country codes; filters ads by country. Case-insensitive.        |
    | platform | The platform through which the ad is shown | string  | Can be "android", "ios", or "web"; filters ads by platform. Case-insensitive.   |

# Examples
* `PUT /api/v1/ad`
  * Create an AD with full conditions
    * Request Body
      ```json
      {
          {
              "title": "AD 55",
              "startAt": "2023-12-10T03:00:00.000Z",
              "endAt": "2023-12-31T16:00:00.000Z",
              "conditions": {
                  "ageStart": 20,
                  "ageEnd": 30,
                  "gender": ["M", "F"],
                  "country": ["TW", "JP"],
                  "platform": ["android", "ios", "web"]
              }
          }
      }
      ```
    * Response with status code 201
      ```json
      {
          "id": 1
      }
      ```
  * Create an AD with an invalid `endAt` argument
    * Request Body
      ```json
      {
          {
              "title": "AD 55",
              "startAt": "2023-12-10T03:00:00.000Z",
              "endAt": "2023-12-09T16:00:00.000Z",          // Should be later than startAt
              "conditions": {
                  "ageStart": 20,
                  "ageEnd": 30,
                  "gender": ["M", "F"],
                  "country": ["TW", "JP"],
                  "platform": ["android", "ios", "web"]
              }
          }
      }
      ```
    * Response with status code 422
      ```json
      {
          "message": "Invalid time frame"
      }
      ```
  * Create an AD with an invalid `ageStart` argument
    * Request Body
      ```json
      {
          {
              "title": "AD 55",
              "startAt": "2023-12-10T03:00:00.000Z",
              "endAt": "2023-12-09T16:00:00.000Z",
              "conditions": {
                  "ageStart": 40,                       // Should be smaller than condition.ageEnd
                  "ageEnd": 30,
                  "gender": ["M", "F"],
                  "country": ["TW", "JP"],
                  "platform": ["android", "ios", "web"]
              }
          }
      }
      ```
    * Response with status code 422
      ```json
      {
          "message": "Invalid start age"
      }
      ```

* `GET /api/v1/ad`
  * Query ADs with full conditions
    * Request `GET /api/v1/ad?offset=10&limit=3&age=24&gender=F&country=TW&platform=ios`
    * Response with status code 200
      ```json
      {
          "ads": [
              {
                  "title": "AD 1",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2024-12-31T16:00:00.000Z"
              },
              {
                  {
                      "title": "AD 10",
                      "startAt": "2023-12-22T01:00:00.000Z",
                      "endAt": "2024-01-31T16:00:00.000Z"
                  }
              }
          ],
          "hasNext": true,
          "nextOffset": 15
      }
      ```
  * Query ADs with no conditions
    * Request `/api/v1/ad?offset=10&limit=3`
    * Response with status code 200
      ```json
      {
          "ads": [
              {
                  "title": "AD 1",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2023-12-31T16:00:00.000Z"
              },
              {
                  "title": "AD 10",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2024-01-31T16:00:00.000Z"
              },
              {
                  "title": "AD 55",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2024-05-31T16:00:00.000Z"
              },
              {
                  "title": "AD 110",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2024-07-31T16:00:00.000Z"
              }
          ],
          "hasNext": true,
          "nextOffset": 15
      }
      ```
  * Query ADs return with no ADs left
    * Request `/api/v1/ad?offset=1000000&limit=50`
    * Response with status code 200
      ```json
      {
          "ads": [
              {
                  "title": "Samsung Galaxy S24",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2024-05-31T16:00:00.000Z"
              },
              {
                  "title": "DTTO Friends",
                  "startAt": "2023-12-22T01:00:00.000Z",
                  "endAt": "2024-07-31T16:00:00.000Z"
              }
          ],
          "hasNext": false,
          "nextOffset": 2147483647
      }
      ```


# Architecture
![System Architecture](image/AD%20system.png)
# Testing Instructions
# Performance Consideration
# Alternative Design
# Contributing Guide