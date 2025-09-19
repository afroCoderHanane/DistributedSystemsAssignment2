# What I see
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 12 57 52 PM" src="https://github.com/user-attachments/assets/69568d2b-c889-4481-a5ad-9023c42cebc9" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 12 57 41 PM" src="https://github.com/user-attachments/assets/56aa69a3-737c-4fbe-881c-9fb22cf0a580" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 12 56 43 PM" src="https://github.com/user-attachments/assets/f5f3005e-e9cc-4e44-b5da-8ef45fdc453a" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 11 50 53 AM" src="https://github.com/user-attachments/assets/a9bf11de-6e3f-4c52-805b-9a7e0bcddc38" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 11 49 55 AM" src="https://github.com/user-attachments/assets/02040d15-edf2-439a-b86e-af364fddd53d" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 11 35 43 AM" src="https://github.com/user-attachments/assets/95d34dd9-7107-40b2-81e6-e2b2d5976b70" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 10 00 53 AM" src="https://github.com/user-attachments/assets/a8020dad-6f98-4759-98d6-226a77195ce3" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 8 32 15 AM" src="https://github.com/user-attachments/assets/69bac4e5-2359-4854-b3fd-8294b4d61a66" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 8 27 53 AM" src="https://github.com/user-attachments/assets/bffc2de6-b89f-4229-a8c1-4797b51ff634" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 8 19 47 AM" src="https://github.com/user-attachments/assets/d52ae42b-4577-4041-8a9c-6117eacbccae" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 8 19 39 AM" src="https://github.com/user-attachments/assets/40a4488e-aa34-44ee-8991-32dc30681c5c" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 7 53 13 AM" src="https://github.com/user-attachments/assets/5fdf2ece-372a-45d4-95be-d3ffaf320361" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 7 52 39 AM" src="https://github.com/user-attachments/assets/5bf45107-3017-481c-b127-3971c374ba9f" />
<img width="1710" height="1107" alt="Screenshot 2025-09-15 at 7 51 03 AM" src="https://github.com/user-attachments/assets/63ce6fd1-7527-44c5-a556-1e50de594283" />

# What is happening in the python script?

```python
import requests

# Correct IP addresses for both instances
EC2_URL1 = "http://35.95.63.45:8080/albums"
EC2_URL2 = "http://44.250.225.59:8080/albums"

POST_DATA = {
    "id": "4",
    "title": "The Modern Sound of Betty Carter",
    "artist": "Betty Carter",
    "price": 49.99
}
HEADERS = {
    "Content-Type": "application/json"
}


def data_from_both(url1, url2):
    try:
        response = requests.get(url1)
        print(f"Instance 1 response: {response.text}")
    except requests.exceptions.RequestException as e:
        print(f"Error occurred while testing {url1}: {e}")

    print("\n\nand...")
    try:
        response = requests.get(url2)
        print(f"Instance 2 response: {response.text}")
    except requests.exceptions.RequestException as e:
        print(f"Error occurred while testing {url2}: {e}")
    return


print("Starting data test...")

data_from_both(EC2_URL1, EC2_URL2)

print("\n\nand adding...")
try:
    response = requests.post(EC2_URL2, json=POST_DATA, headers=HEADERS)
    print(f"Instance 2 response: {response.text}")
except requests.exceptions.RequestException as e:
    print(f"Error occurred while testing {EC2_URL2}: {e}")

data_from_both(EC2_URL1, EC2_URL2)

print("uhoh... what happened?")
```
This script demonstrates that the two EC2 instances are completely independent and do not share data or state with each other.
When the script runs, the new album is successfully added, but only to the second instance (EC2_URL2). The first instance (EC2_URL1) remains unchanged. This leads to the two instances reporting different sets of data at the end of the script.
What Happens: A Step-by-Step Breakdown
* Initial State Check: The first call to data_from_both(EC2_URL1, EC2_URL2) sends a GET request to both instances. Since they were just created from the same source, they will return the exact same initial list of albums.
* Adding Data to One Instance: The script then sends a POST request containing the "Betty Carter" album. Crucially, this request is sent only to EC2_URL2. This action modifies the data held in memory by the web service running on the second instance only. The first instance is completely unaware of this change.
* Final State Check: The second call to data_from_both(EC2_URL1, EC2_URL2) reveals the inconsistency:
* The GET request to Instance 1 returns the original, default list of albums.
* The GET request to Instance 2 returns the original list plus the new "Betty Carter" album.
Why It Happens: Independent and Stateless Instances
The core reason for this behavior is that each EC2 instance is an isolated virtual server. Think of them as two separate computers. Each one has its own memory and runs its own copy of the web service application.
* No Shared Memory: The album data is being stored in the memory of the running application on each instance. When you POST new data to one instance, you are only changing the contents of its memory. There is no automatic process that copies or synchronizes this memory with any other instance.
* Stateless Architecture: From the perspective of an external system, each HTTP request is an independent transaction. The instances don't inherently know about each other or the requests that other instances have handled.
This experiment highlights a fundamental challenge in scaling applications. If you have multiple servers (for handling more traffic, for example), you can't store changing data in the server's local memory. If you did, users would get inconsistent results depending on which server their request was routed to. The solution is to use a shared, external data store, like a database (e.g., Amazon RDS, DynamoDB), that all instances can read from and write to.
