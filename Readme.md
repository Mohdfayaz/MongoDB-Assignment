Database Schema Design;

users;

json
{
  "_id": ObjectId(),
  "name": "Mohammed Fayaz",
  "email": "mdfoe123@gmail.com",
  "batch": "B11",
  "mentor_id": ObjectId(),
  "codekata_score": 120,
  "attendance": [
    { "date": "2020-10-15", "status": "present" },
    { "date": "2020-10-16", "status": "absent" }
  ],
  "tasks": [
    { "task_id": ObjectId(), "submitted": true, "date": "2020-10-15" }
  ],
  "placement_drives": [ ObjectId() ]  // Reference to company_drives
}


codekata;

json
{
  "_id": ObjectId(),
  "user_id": ObjectId(),
  "problems_solved": 150
}


attendance;

json
{
  "_id": ObjectId(),
  "user_id": ObjectId(),
  "date": "2020-10-15",
  "status": "present"
}

topics;

json
{
  "_id": ObjectId(),
  "topic_name": "MongoDB",
  "date": "2020-10-10"
}

tasks;

json
{
  "_id": ObjectId(),
  "task_name": "Build a REST API",
  "date": "2020-10-12",
  "submitted_by": [ ObjectId() ]  // List of user IDs who submitted
}

company_drives;

json
{
  "_id": ObjectId(),
  "company_name": "Google",
  "drive_date": "2020-10-20",
  "appeared_users": [ ObjectId() ] // List of user IDs
}

mentors;

json
{
  "_id": ObjectId(),
  "name": "Pugazharasan",
  "mentees": [ ObjectId(), ObjectId() ]  // List of user IDs
}

Queries;

1. Find all topics and tasks taught in October 2020;

js
db.topics.find({ 
  date: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } 
});

db.tasks.find({ 
  date: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } 
});

2. Find all company drives between 15-Oct-2020 and 31-Oct-2020;

js
db.company_drives.find({
  drive_date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") }
});

3. Find all company drives and students who appeared for placements;

js
db.company_drives.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "appeared_users",
      foreignField: "_id",
      as: "students"
    }
  }
]);

4. Find the number of problems solved by each user in Codekata;

js
db.codekata.find({}, { user_id: 1, problems_solved: 1 });

5. Find all mentors who have more than 15 mentees;

js
db.mentors.find({ 
  $expr: { $gt: [{ $size: "$mentees" }, 15] } 
});

6. Find the number of users who were absent and did not submit tasks between 15-Oct-2020 and 31-Oct-2020;

js
db.users.aggregate([
  {
    $match: {
      attendance: {
        $elemMatch: { date: { $gte: "2020-10-15", $lte: "2020-10-31" }, status: "absent" }
      },
      tasks: {
        $not: {
          $elemMatch: { date: { $gte: "2020-10-15", $lte: "2020-10-31" }, submitted: true }
        }
      }
    }
  },
  {
    $count: "absent_users_without_tasks"
  }
]);




OUTPUT - - - -




1. Find all topics and tasks taught in October 2020;

Query;

js
db.topics.find({ 
  date: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } 
});

js
db.tasks.find({ 
  date: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } 
});

 -- -- -- Output;

json
[
  { "_id": ObjectId("1"), "topic_name": "MongoDB", "date": "2020-10-10" },
  { "_id": ObjectId("2"), "topic_name": "Node.js", "date": "2020-10-15" }
]
json
[
  { "_id": ObjectId("1"), "task_name": "Build a REST API", "date": "2020-10-12" },
  { "_id": ObjectId("2"), "task_name": "React UI Design", "date": "2020-10-20" }
]

2. Find all company drives between 15-Oct-2020 and 31-Oct-2020;

Query;

js
db.company_drives.find({
  drive_date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") }
});

 Output;

json
[
  { "_id": ObjectId("1"), "company_name": "Google", "drive_date": "2020-10-20" },
  { "_id": ObjectId("2"), "company_name": "Amazon", "drive_date": "2020-10-25" }
]

3. Find all company drives and students who appeared for placements;

Query;

js
db.company_drives.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "appeared_users",
      foreignField: "_id",
      as: "students"
    }
  }
]);

 Output;

json
[
  {
    "_id": ObjectId("1"),
    "company_name": "Google",
    "drive_date": "2020-10-20",
    "students": [
      { "_id": ObjectId("101"), "name": "Fayaz" },
      { "_id": ObjectId("102"), "name": "Taufiq" }
    ]
  },
  {
    "_id": ObjectId("2"),
    "company_name": "Amazon",
    "drive_date": "2020-10-25",
    "students": [
      { "_id": ObjectId("103"), "name": "Noor" }
    ]
  }
]

4. Find the number of problems solved by each user in Codekata;

Query;
js
db.codekata.find({}, { user_id: 1, problems_solved: 1 });
 
 Output;

json
[
  { "user_id": ObjectId("101"), "problems_solved": 150 },
  { "user_id": ObjectId("102"), "problems_solved": 200 }
]

5. Find all mentors who have more than 15 mentees;

Query;
js
db.mentors.find({ 
  $expr: { $gt: [{ $size: "$mentees" }, 15] } 
});

 Output;

json
[
  { "_id": ObjectId("1"), "name": "Pugazharasan", "mentees": [ObjectId("101"), ObjectId("102"), ... 16 more] }
]

6. Find the number of users who were absent and did not submit tasks between 15-Oct-2020 and 31-Oct-2020;

Query;

js
db.users.aggregate([
  {
    $match: {
      attendance: {
        $elemMatch: { date: { $gte: "2020-10-15", $lte: "2020-10-31" }, status: "absent" }
      },
      tasks: {
        $not: {
          $elemMatch: { date: { $gte: "2020-10-15", $lte: "2020-10-31" }, submitted: true }
        }
      }
    }
  },
  {
    $count: "absent_users_without_tasks"
  }
]);

 Output;

json
[
  { "absent_users_without_tasks": 5 }
]