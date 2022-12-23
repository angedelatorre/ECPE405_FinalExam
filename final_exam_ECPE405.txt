const express = require("express");
const bodyParser = require("body-parser");
const app = express();
const { MongoClient, ObjectId } = require("mongodb"); //https://github.com/mongodb/node-mongodb-native
const { isObjectIdOrHexString} = require("mongoose");
const port = 5050;

// Set up default mongoose connection
const url = "mongodb+srv://associate-developer:angel1102@cluster0.59dojsa.mongodb.net/test";
const client = new MongoClient(url);

app.use(
  bodyParser.urlencoded({
    extended: false,
  })
);

const dbName = "JAR_Clinic";
let db;
client
  .connect()
  .then(async () => {
    db = client.db(dbName);
    console.log("Connected to Mongodb");
  })
  .catch((err) => {
    console.log(err);
    console.log("Unable to connect to Mongodb");
  });

  //GET ALL Patient
app.get("/Patients", (req, res) => {
  db.collection("Patients")
    .find({})
    .toArray()
    .then((records) => {
      return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Proces" });
  });
});

//1. POST Patient //Create a patient
app.post("/Patients", (req, res) => {
  console.log(req.body);
  const {Patient_ID, Patient_Name, Gender, Age, Date_of_Birth, Address, Contact_Number} = req.body;
  db.collection("Patients")
    .insertOne({
      Patient_ID,
      Patient_Name,
      Gender,
      Age,
      Date_of_Birth,
      Address,
      Contact_Number
    })
  .then((records) => {
      return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Process" });
  });
});


//2. PUT Patient //Update information of the patient
app.put("/Patients/:Patient_ID", (req, res) => {
  const Patient_ID = req.params.Patient_ID;
  const Patient_Name = req.body.Patient_Name;
  const Gender = req.body.Gender;
  const Age = req.body.Age;
  const Date_of_Birth = req.body.Date_of_Birth;
  const Address = req.body.Address;
  const Contact_Number = req.body.Contact_Number;
  db.collection("Patients")
  .updateOne(
    {
      Patient_ID
    },
    {
      $set: {
        Patient_Name,
        Gender,
        Age,
        Date_of_Birth,
        Address,
        Contact_Number
      }
    })
  .then((records) => {
    return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Proces" });
  });
});

//3. DELETE Patient
app.delete("/Patients/:Patient_ID", (req, res) => {
  const Patient_ID = req.params.Patient_ID;
    db.collection("Patients")
    .deleteOne(
      {
        Patient_ID
      }
    )
  .then((records) => {
    return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Proces" });
  });
});


//4. POST Prescription 
//Attach a prescription to the patient
app.post("/Prescriptions", (req, res) => {
  console.log(req.body);
  const {Patient_ID, Diagnose, Date_Visit, Doctor, Medications} = req.body;
  db.collection("Prescriptions")
    .insertOne({
      Patient_ID,
      Diagnose,
      Date_Visit,
      Doctor,
      Medications
    }
  )
  .then((records) => {
      return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Process" });
  });
});

//5. ADD MEDICATION TO PRESCRIPTION
app.put("/Prescriptions/:_id", (req, res) => {
  const id = req.params._id;
  const {Medications} = req.body;
  db.collection("Prescriptions")
  .updateOne(
    {
      _id : ObjectId(id)
    },
    {
      $push: {
        Medications
      }
    }
  )
  .then((records) => {
    return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Proces" });
  });
});

//6.REMOVE MEDICATION
app.delete("/Prescriptions/:_id", (req, res) => {
  const id = req.params._id;
  const {Medications} = req.body;
    db.collection("Prescriptions")
    .updateOnes(
      {
        _id: ObjectId(id)
      },
      {
        $pull:{
          Medications
        }
      }
    )
  .then((records) => {
    return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Proces" });
  });
});

//7. LIST OF MEDICATION
app.get("/Prescriptions/:_id", (req, res) => {
  const id = req.params._id;
  db.collection("Prescriptions")
    .find({_id: ObjectId(id)})
    .toArray()
    .then((records) => {
      return res.json(records);
    })
    .catch((err) => {
      console.log(err);
      return res.json({ msg: "There was an error processing your query" });
    });
}); 

//8. DELETE PRESCRIPTION
app.delete("/Prescriptions/:_id", (req,res) => {
  const id = req.params._id;
    db.collection("Prescriptions")
    .deleteOne(
      {
        _id: ObjectId(id)
      }
    )
  .then((records) => {
    return res.json(records);
  })
  .catch((err) => {
    console.log(err);
    return res.json({ msg: "ERROR on Query Proces" });
  });
});

app.listen(port, () => {
 console.log(`Example app listening on port ${port}`);
});