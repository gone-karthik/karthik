const express = require('express')
const path = require('path')

const {open} = require('sqlite')
const sqlite3 = require('sqlite3')
const app = express()

const dbPath = path.join(__dirname, 'cricketTeam.db')

let db = null

const initializeDBAndServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite3.Database,
    })
    app.listen(3000, () => {
      console.log('Server Running at http://localhost:3000/')
    })
  } catch (e) {
    console.log(`DB Error: ${e.message}`)
    process.exit(1)
  }
}

initializeDBAndServer()

app.get('/players/', async (request, response) => {
  const getPlayerQuery = `
    SELECT
      *
    FROM
      book
    ORDER BY
      playerId;`
  const playerArray = await db.all(getPlayerQuery)
  response.send(playerArray)
})

app.post('/players/', async (request, response) => {
  const bookDetails = request.body
  const {playerId, playerName, jerseyNumber, role} = bookDetails
  const addBookQuery = `
    INSERT INTO
      players ( player_Id,
          player_Name,
       jersey_Number,
       role)
    VALUES
      (
        ${playerId},
         '${playerName}',
         ${jerseyNumber},
         '${role}'
      );`

  const dbResponse = await db.run(addBookQuery)
  const bookId = dbResponse.lastID
  response.send({bookId: bookId})
})

app.get('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params
  const getPlayerQuery = `
    SELECT
      *
    FROM
      book
    WHERE
      book_id = ${playerId};`
  const player = await db.get(getPlayerQuery)
  response.send(player)
})

app.put('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params
  const playerDetails = request.body
  const {playerId, playerName, jerseyNumber, role} = playerDetails
  const updatePlayerQuery = `
    UPDATE
      book
    SET
     player_Id=${playerId},
     player_name='${playerName},
     jersey_number=${jerseyNumber},
     role='${role}'
     
    WHERE
      player_id = ${playerId};`
  await db.run(updatePlayerQuery)
  response.send('Player Details Updated')
})

app.delete('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params
  const deletePlayerQuery = `
    DELETE FROM
      players
    WHERE
      player_id = ${playerId};`
  await db.run(deletePlayerQuery)
  response.send('Player Removed')
})

module.exports = app
# karthik
