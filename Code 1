const express = require('express');
const cors = require('cors');
const { Configuration, OpenAIApi } = require('openai');
const { Pool } = require('pg');
const axios = require('axios');
require('dotenv').config();
const app = express();
app.use(cors());
app.use(express.json());

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

const openai = new OpenAIApi(new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
}));

app.get('/api/todos', async (req, res) => {
  const result = await pool.query('SELECT * FROM todos ORDER BY id DESC');
  res.json(result.rows);
});

app.post('/api/todos', async (req, res) => {
  const { text } = req.body;
  await pool.query('INSERT INTO todos (text) VALUES ($1)', [text]);
  res.sendStatus(201);
});

app.delete('/api/todos/:id', async (req, res) => {
  await pool.query('DELETE FROM todos WHERE id = $1', [req.params.id]);
  res.sendStatus(204);
});

app.post('/api/summarize', async (req, res) => {
  const result = await pool.query('SELECT * FROM todos');
  const todos = result.rows.map(t => `- ${t.text}`).join('\n');

  const completion = await openai.createChatCompletion({
    model: 'gpt-3.5-turbo',
    messages: [{ role: 'user', content: `Summarize these todos:\n${todos}` }],
  });

  const summary = completion.data.choices[0].message.content;
  await axios.post(process.env.SLACK_WEBHOOK_URL, { text: summary });

  res.sendStatus(200);
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
