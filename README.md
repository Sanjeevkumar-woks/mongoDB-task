
# MongoDB Aggregation and Query Examples


## 1. Find All the Topics and Tasks Taught in October

This query fetches all the topics and their associated tasks that were taught during October 2020.

```javascript
db.getCollection('Topics').aggregate(
  [
    {
      $lookup: {
        from: 'Tasks',
        localField: 'topic',
        foreignField: 'task',
        as: 'Topic Task Data'
      }
    },
    {
      $match: { topicDate: { $regex: '2020-10' } }
    },
    {
      $project: {
        _id: 0,
        topic: 1,
        topicDate: 1,
        'Topic Task Data.userId': 1,
        'Topic Task Data.submitted': 1,
        'Topic Task Data.task': 1
      }
    }
  ] 
);

```

## 2. Find All the Company Drives Between 15-Oct-2020 and 31-Oct-2020
This query returns all the company drives that occurred between October 15, 2020, and October 31, 2020.

```
db.getCollection('Company_Drives').find({
  driveDate: {
    $gte: '2020-10-15',
    $lte: '2020-10-31'
  }
});

```

## 3. Find All the Company Drives and Students Who Appeared for Placement
This aggregation returns all the company drives along with the students who appeared for the placement.

```
db.getCollection('Company_Drives').aggregate(
  [
    {
      $lookup: {
        from: 'Users',
        localField: 'userId',
        foreignField: 'userId',
        as: 'studentInfo'
      }
    },
    {
      $project: {
        _id: 0,
        'studentInfo.userName': 1,
        company: 1,
        driveDate: 1,
        'studentInfo.userEmail': 1,
        'studentInfo.userId': 1
      }
    }
  ],
  
);
```

##  4. Find the Total Number of Problems Solved by Users in Codekata
This query aggregates the total number of problems solved by all users in Codekata.

```
db.getCollection('Codekata').aggregate(
  [
    {
      $group: {
        _id: 'Total Problem Solved By Users',
        count: { $sum: '$problemSolved' }
      }
    }
  ]
);
```

## 5. Find All the Mentors with More Than 15 Mentees
This query returns all mentors who have more than 15 mentees.

```
db.getCollection('Mentors').aggregate(
  [{ $match: { menteeCount: { $gt: 15 } } }]
);

```

## 6. Find the Number of Users Absent and Task Not Submitted Between 15-Oct-2020 and 31-Oct-2020
This query fetches the number of users who were absent and did not submit their tasks between October 15, 2020, and October 31, 2020.

```
db.getCollection('Attendence').aggregate(
  [
    {
      $lookup: {
        from: 'Topics',
        localField: 'topicId',
        foreignField: 'topicId',
        as: 'Absent'
      }
    },
    {
      $lookup: {
        from: 'Tasks',
        localField: 'userId',
        foreignField: 'userId',
        as: 'Task-notSubmitted'
      }
    },
    { $unwind: '$Task-notSubmitted' },
    { $unwind: '$Absent' },
    {
      $match: {
        attended: false,
        'Absent.topicDate': {
          $gte: '2020-10-15',
          $lte: '2020-10-31'
        }
      }
    },
    {
      $match: {
        'Task-notSubmitted.dueDate': {
          $gte: '2020-10-15',
          $lte: '2020-10-31'
        },
        'Task-notSubmitted.submitted': false
      }
    },
    { $project: { _id: 0, 'Absent._id': 0 } },
    { $project: { 'task-notSubmitted._id': 0 } }
  ]
);
```
