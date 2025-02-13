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
