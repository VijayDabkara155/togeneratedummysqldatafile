# togeneratedummysqldatafile
togeneratedummysqldatafile
in linux bash

#!/bin/bash

# Output file
OUTFILE="dummy_data.sql"
TARGET_SIZE_MB=210
TARGET_SIZE_BYTES=$((TARGET_SIZE_MB * 1024 * 1024))
BATCH_SIZE=1000

# Start fresh
rm -f "$OUTFILE"

# Write schema
cat <<EOF >> "$OUTFILE"
-- Dummy SQL File
DROP DATABASE IF EXISTS DummyDB;
CREATE DATABASE DummyDB;
USE DummyDB;

CREATE TABLE Users (
    UserID INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(100),
    Email VARCHAR(100),
    SignupDate DATE
);

EOF

echo "Generating dummy INSERTs to reach $TARGET_SIZE_MB MB..."

# Function to generate one batch of inserts
generate_insert_batch() {
    for ((i=1; i<=BATCH_SIZE; i++)); do
        NAME="User$RANDOM"
        EMAIL="user${RANDOM}@example.com"
        SIGNUP_DATE="$(shuf -i 2015-01-01-2025-12-31 -n 1 | tr '-' '/')"
        echo -n "('$NAME', '$EMAIL', '$(date -d $SIGNUP_DATE +%F)')" >> "$OUTFILE"
        if [[ $i -lt $BATCH_SIZE ]]; then
            echo -n "," >> "$OUTFILE"
        else
            echo ";" >> "$OUTFILE"
        fi
    done
}

# Insert loop
echo -n "INSERT INTO Users (Name, Email, SignupDate) VALUES" >> "$OUTFILE"

while [[ $(stat -c%s "$OUTFILE") -lt $TARGET_SIZE_BYTES ]]; do
    generate_insert_batch
    echo -n "INSERT INTO Users (Name, Email, SignupDate) VALUES" >> "$OUTFILE"
    printf "\rCurrent size: $(($(stat -c%s "$OUTFILE") / 1024 / 1024)) MB"
done

echo -e "\nDone! File: $OUTFILE (Size: $(($(stat -c%s "$OUTFILE") / 1024 / 1024)) MB)"

