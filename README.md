#!/bin/bash

OUTFILE="dummy_data.sql"
TARGET_SIZE_MB=210
TARGET_SIZE_BYTES=$((TARGET_SIZE_MB * 1024 * 1024))
BATCH_SIZE=1000
MAX_LOOPS=10000
counter=0

echo "DROP DATABASE IF EXISTS DummyDB;" > $OUTFILE
echo "CREATE DATABASE DummyDB;" >> $OUTFILE
echo "USE DummyDB;" >> $OUTFILE
echo "CREATE TABLE Users (UserID INT AUTO_INCREMENT PRIMARY KEY, Name VARCHAR(100), Email VARCHAR(100), SignupDate DATE);" >> $OUTFILE

while [ $(stat -c%s "$OUTFILE") -lt $TARGET_SIZE_BYTES ] && [ $counter -lt $MAX_LOOPS ]
do
  SQL="INSERT INTO Users (Name, Email, SignupDate) VALUES "
  for ((i=1; i<=BATCH_SIZE; i++))
  do
    NAME="User$RANDOM"
    EMAIL="user$RANDOM@example.com"
    SIGNUPDATE="2025-01-01"
    SQL+="('$NAME','$EMAIL','$SIGNUPDATE')"
    if [ $i -lt $BATCH_SIZE ]; then
      SQL+=","
    else
      SQL+=";"
    fi
  done
  echo $SQL >> $OUTFILE
  ((counter++))
  echo "Iteration: $counter, File size: $(du -h $OUTFILE | cut -f1)"
done

echo "Done generating dummy SQL file."
