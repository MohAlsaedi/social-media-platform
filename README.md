# MongoDB Social Media Platform

This project is a simple social media platform built with MongoDB to simulate how social networking applications work. It demonstrates data modeling, relationships between collections, and data retrieval using MongoDB operations and aggregations.

## Prerequisites

Before working with this project, make sure you have:

* MongoDB Community Server installed
* MongoDB Compass
* Basic understanding of MongoDB collections and documents
* Knowledge of Primary Keys and Foreign Keys concepts

## Project Structure

### Users Collection

Stores user information.

| Field    | Description          |
| -------- | -------------------- |
| `_id`    | Primary Key (PK)     |
| `name`   | User's name          |
| `age`    | User's age           |
| `email`  | User's email address |
| `gender` | User's gender        |

### Follow Collection

Stores follow relationships between users.

| Field       | Description                                       |
| ----------- | ------------------------------------------------- |
| `_id`       | Document ID                                       |
| `user_id`   | Foreign Key (FK) referencing the Users collection |
| `following` | ID of the user being followed                     |
| `follow`    | ID of the follower                                |

**Note:** The `following` and `follow` fields reference user IDs from the Users collection, allowing relationship queries and joins using MongoDB aggregation.

### Posts Collection

Stores user posts.

| Field     | Description                                       |
| --------- | ------------------------------------------------- |
| `_id`     | Document ID                                       |
| `user_id` | Foreign Key (FK) referencing the Users collection |
| `title`   | Post title                                        |
| `content` | Post content                                      |

**Note:** The `user_id` field references the Users collection, making it possible to retrieve author information and connect posts to their creators through aggregation pipelines.

## Sample Aggregation Query

The following aggregation pipeline demonstrates how to use multiple `$lookup` operations to retrieve a user's followers and following relationships from the **Users** collection.

```javascript
db.follow.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "user_id",
      foreignField: "_id",
      as: "user"
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "following",
      foreignField: "_id",
      as: "following_users"
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "follower",
      foreignField: "_id",
      as: "followers"
    }
  },
  {
    $project: {
      _id: 0,
      user: { $arrayElemAt: ["$user.name", 0] },
      following: "$following_users.name",
      followers: "$followers.name"
    }
  }
]);
```

### Sample Output

```javascript
{
  user: "Sara",
  following: ["Ahmed", "Noura"],
  followers: ["Noura", "Yasser"]
},
{
  user: "Noura",
  following: ["Yasser", "Sara"],
  followers: ["Ahmed", "Noura"]
},
{
  user: "Yasser",
  following: ["Noura", "Sara", "Ahmed"],
  followers: ["Noura", "Ahmed"]
},
{
  user: "Ahmed",
  following: ["Yasser"],
  followers: ["Sara", "Yasser"]
}
```

### Explanation

* The first `$lookup` retrieves the user's information from the **Users** collection.
* The second `$lookup` retrieves the users that the current user is following.
* The third `$lookup` retrieves the users who follow the current user.
* The `$project` stage formats the output by:

  * Hiding the `_id` field.
  * Displaying the user's name instead of the user ID.
  * Returning arrays of follower and following names.

This query demonstrates how MongoDB's Aggregation Framework can be used to simulate relationships and perform joins between collections, similar to relational databases.



