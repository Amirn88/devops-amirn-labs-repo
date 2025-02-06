Determine one by one if the ID number of the user (last column) is odd or even number if it's specified.
● Check whether that user has a routable, FQDN email address. If so, write a message to stdout like:
The $ID of $EMAIL is (even|odd) number.
If the user has no valid email address OR user ID, the script should silently
continue the next user showing a warning for the invalid parameters
The content of the source text file:
name, email, id
John, john_j123@domain.com, 325
Susan, ,510
Menna, menna_ka234@domain.com
Magdy, magdy_mm23@localhost, 150
Jake, jake_j121@domain.com, 131

____---------__________------__-----__---_--_--___--_-_--_______-_--_-
Answer:
------------
vi users.txt
"
John, john_j123@domain.com, 325
Susan, ,510
Menna, menna_ka234@domain.com
Magdy, magdy_mm23@localhost, 150
Jake, jake_j121@domain.com, 131
"
sudo vi app.sh

#!/bin/bash

# Path to the input file
input_file="users.txt"

# Read the file line by line, skipping the header
tail -n +2 "$input_file" | while IFS=, read -r name email id; do
  # Trim leading/trailing whitespace from fields
  name=$(echo "$name" | xargs)
  email=$(echo "$email" | xargs)
  id=$(echo "$id" | xargs)

  # Skip if ID is empty or not a number
  if [[ -z "$id" || ! "$id" =~ ^[0-9]+$ ]]; then
    echo "Warning: Invalid or missing ID for user '$name'. Skipping." >&2
    continue
  fi

  # Skip if email is empty or not a valid FQDN email
  if [[ -z "$email" || ! "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
    echo "Warning: Invalid or missing email for user '$name'. Skipping." >&2
    continue
  fi

  # Determine if the ID is odd or even
  if (( id % 2 == 0 )); then
    parity="even"
  else
    parity="odd"
  fi

  # Print the result
  echo "The $id of $email is $parity number."
done


----------------------------------------
chmod +x app.sh
./app.sh
