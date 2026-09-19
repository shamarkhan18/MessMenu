# 🍽️ MessMenu

**Know what's cooking. Rate what you eat.**

MessMenu is a serverless web app that helps hostel/mess students check daily meal menus and rate individual meals, with live aggregated feedback trends — built for the **WeMakeDevs Build It Hackathon**.

---

## 🧩 Problem It Solves

Hostel mess menus are often shared informally through WhatsApp groups, printed notices, or word of mouth. Students may have no convenient way to access the menu or provide structured feedback about individual meals.

MessMenu provides a simple web app where students can:

* Check the menu for a specific date
* Rate individual meals such as breakfast, lunch, snacks, and dinner
* Add optional comments with their ratings
* View live rating trends for meals

This creates a simple feedback loop between students and mess management.

---

## ✨ Features

* 📅 **Date-based menu browsing** — navigate between days and view the corresponding menu
* ⭐ **Meal-wise star ratings** — rate individual meals with an optional comment
* 📊 **Live trends** — view average ratings and total rating counts for individual meals
* 🚀 **Serverless architecture** — built using AWS managed services without a traditional backend server
* 💾 **Persistent storage** — menu, rating, and trend data are stored in DynamoDB

---

## 🏗️ AWS Architecture

```text
                 ┌──────────────────┐
                 │     Student      │
                 │    Web Browser   │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    Amazon S3     │
                 │ Static Frontend  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │   API Gateway    │
                 │    HTTP API      │
                 └────────┬─────────┘
                          │
             ┌────────────┼────────────┐
             ▼            S▼            ▼
       ┌──────────┐ ┌──────────┐ ┌──────────┐
       │  Menu    │ │ Ratings  │ │  Trends  │
       │  Lambda  │ │  Lambda  │ │  Lambda  │
       └────┬─────┘ └────┬─────┘ └────┬─────┘
            │             │             │
            ▼             ▼             ▼
       ┌──────────┐ ┌──────────┐ ┌──────────┐
       │ MessMenu │ │MessRatings│ │MessTrends│
       │ DynamoDB │ │ DynamoDB │ │ DynamoDB │
       └──────────┘ └──────────┘ └──────────┘
```

---

## 🛠️ Tech Stack

| Layer    | Technology                       |
| -------- | -------------------------------- |
| Frontend | HTML, CSS, Vanilla JavaScript    |
| Hosting  | Amazon S3 Static Website Hosting |
| API      | Amazon API Gateway HTTP API      |
| Compute  | AWS Lambda (Python)              |
| Database | Amazon DynamoDB                  |

---

## 🔌 API Endpoints

| Method | Route                                 | Description                                                             |
| ------ | ------------------------------------- | ----------------------------------------------------------------------- |
| `GET`  | `/menu?date=YYYY-MM-DD`               | Fetch the menu for a given date                                         |
| `POST` | `/menu`                               | Add or update a menu for a date                                         |
| `POST` | `/ratings`                            | Submit a rating for a specific date and meal                            |
| `GET`  | `/ratings?dateMeal=YYYY-MM-DD%23meal` | Retrieve ratings for a specific meal                                    |
| `GET`  | `/trends?dateMeal=YYYY-MM-DD%23meal`  | Calculate the average rating and total rating count for a specific meal |

> `#` in `dateMeal` is URL-encoded as `%23` when sent through the API.

---

## ⚙️ How It Works

1. The frontend is hosted as a static website on **Amazon S3**.
2. The browser communicates directly with **Amazon API Gateway** using JavaScript `fetch()` requests.
3. `GET /menu` triggers **messMenuHandler**, which retrieves the requested day's menu from the `MessMenu` DynamoDB table using `date` as the partition key.
4. `POST /ratings` triggers **messRatingsHandler**, which stores the student's rating in the `MessRatings` DynamoDB table.
5. The `MessRatings` table uses `dateMeal` as its partition key and `studentId` as its sort key, allowing a student to maintain one rating per meal for a given date.
6. `GET /trends` triggers **messTrendsHandler**, which retrieves ratings for the requested meal, calculates the average rating and total number of ratings, and stores the resulting trend in `MessTrends`.
7. The calculated trend is returned to the frontend and displayed immediately.
8. The Lambda functions are stateless and the application has no traditional always-running backend server.

---

## 🗄️ DynamoDB Design

### MessMenu

| Attribute   | Role          |
| ----------- | ------------- |
| `date`      | Partition Key |
| `breakfast` | Meal data     |
| `lunch`     | Meal data     |
| `snacks`    | Meal data     |
| `dinner`    | Meal data     |

### MessRatings

| Attribute   | Role              |
| ----------- | ----------------- |
| `dateMeal`  | Partition Key     |
| `studentId` | Sort Key          |
| `rating`    | Rating value      |
| `comment`   | Optional feedback |

### MessTrends

| Attribute       | Role               |
| --------------- | ------------------ |
| `dateMeal`      | Partition Key      |
| `averageRating` | Calculated average |
| `totalRatings`  | Number of ratings  |

---

## 🔭 Future Scope

* **Admin panel** for mess staff to add and edit menus
* **Amazon Cognito authentication** for verified student accounts
* **Push notifications** for daily menu updates
* **Historical analytics dashboard** for longer-term meal trends
* **DynamoDB Streams + Lambda** for event-driven precomputation of trend data at larger scale

---

## 🎥 Demo Flow

1. Open the MessMenu web app.
2. Navigate to a date with available menu data.
3. View breakfast, lunch, snacks, and dinner.
4. Select a meal and submit a star rating with an optional comment.
5. Observe the Trends section update with the new average and rating count.
6. Refresh the page to verify that the data persists.
7. Explain that the request flows through **S3 → API Gateway → Lambda → DynamoDB** without a traditional backend server.

---

## 📦 Deployment

### Frontend

The frontend is deployed as a static website using **Amazon S3 Static Website Hosting**.

### Backend

The backend consists of three independent AWS Lambda functions:

* `messMenuHandler`
* `messRatingsHandler`
* `messTrendsHandler`

They are exposed through an **Amazon API Gateway HTTP API** using Lambda proxy integration.

The Lambda functions access DynamoDB through their IAM execution roles.

---

## 🚀 Live Demo

**MessMenu:**
http://messmenu-api-samk18.s3-website-us-east-1.amazonaws.com

---

Built for the **WeMakeDevs Build It Hackathon**.
