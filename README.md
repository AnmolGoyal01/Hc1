const express = require('express');
const mongoose = require('mongoose');
const multer = require('multer');
const csv = require('csv-parser');
const fs = require('fs');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/bankDB', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Define Mongoose Models
const Bank = mongoose.model('Bank', new mongoose.Schema({
  bic: String,
  charge: Number,
}));

const Link = mongoose.model('Link', new mongoose.Schema({
  fromBic: String,
  toBic: String,
  time: Number,
}));

const app = express();
const upload = multer({ dest: 'uploads/' });

// Helper function to parse and save CSV
const processCSV = (filePath, Model) => {
  return new Promise((resolve, reject) => {
    const results = [];
    fs.createReadStream(filePath)
      .pipe(csv())
      .on('data', (data) => results.push(data))
      .on('end', async () => {
        try {
          await Model.insertMany(results);
          fs.unlinkSync(filePath);
          resolve('Upload successful');
        } catch (error) {
          reject(error);
        }
      })
      .on('error', (error) => reject(error));
  });
};

// Routes to upload files
app.post('/upload/banks', upload.single('file'), async (req, res) => {
  try {
    const message = await processCSV(req.file.path, Bank);
    res.status(200).json({ message });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/upload/links', upload.single('file'), async (req, res) => {
  try {
    const message = await processCSV(req.file.path, Link);
    res.status(200).json({ message });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => console.log('Server running on port 3000'));
