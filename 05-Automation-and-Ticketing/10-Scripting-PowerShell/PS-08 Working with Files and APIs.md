---
tags: [powershell, scripting, files, api, automation]
aliases: [PS-08, PowerShell Files, PowerShell API]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-purple]
> 📜 **POWERSHELL**

`#complete` `#intermediate` `#none`

# PS-08: Working with Files and APIs

> [!abstract] Overview
> *PowerShell mein files (TXT, CSV, JSON) aur APIs ke saath kaam karna ek core automation skill hai.* Yeh note cover karta hai ki kaise aap files read/write kar sakte hain aur REST APIs se data fetch karke usko process kar sakte hain. Ek support engineer ke liye yeh scripting tasks jaise log analysis, user provisioning, aur monitoring tools integration ke liye bahut zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Reading, writing, and manipulating local files, as well as interacting with external web services via APIs using PowerShell.
- **Why it matters** — *System administration mein har cheez manual nahi ki ja sakti.* Automation requires reading data from files (like lists of users), saving reports, and calling web APIs (like Microsoft Graph or ticketing systems) to automate workflows.
- **Where you see this** — Processing CSV files to create Active Directory users, downloading JSON configuration files, querying REST API to check service health, parsing system logs.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Run existing scripts to generate CSV reports or check basic API endpoints. |
| **L2** | Modify scripts to parse files, update API authentication tokens, format outputs. |
| **L3** | Architect complex integrations, build robust API error handling, optimize file processing for large datasets. |

> [!tip] Seedha Simple Mein
> *Files se data nikalna ya API se data mangwana PowerShell ko external duniya se connect karta hai. Jaise tum Excel mein data padhte ho, waise hi PS file padhta hai, aur API ek online web service hai jo tumhe directly data deti hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Working with Files and APIs** is like **Ordering Food and Managing Receipts** because...
>
> - **Reading/Writing Files** is like checking your physical receipt or writing down a grocery list. You are dealing with data saved locally.
> - **Calling an API** is like calling a restaurant's drive-thru or using a food delivery app. You send a request ("give me a pizza"), and the restaurant processes it and sends back a response (the food, or an error if they are out of stock). You don't care how the kitchen works, you just care about the result.

---
## 🔬 Technical Deep Dive

### 1. Working with Text and CSV Files

> [!info] Key Concept
> File I/O operations are fundamental. PowerShell uses cmdlets like `Get-Content`, `Set-Content`, `Import-Csv`, and `Export-Csv` to handle data formats easily.

**Text Files:**
- `Get-Content`: Reads a file line by line. *Ek ek line read karta hai.*
- `Out-File` or `Set-Content`: Writes data to a file. `>>` operator also appends.

**CSV Files:**
PowerShell is highly optimized for CSV files. `Import-Csv` converts each row into a PowerShell object, making it easy to filter and manipulate.
- *CSV files mein columns properties ban jaate hain.*

> [!danger] Common Mistake
> Using `Set-Content` instead of `Add-Content` when trying to append data. `Set-Content` will overwrite the entire file! *Dhyan rakhna, Set-Content purana data delete kar dega.*

### 2. Handling JSON Data

> [!info] Key Concept
> JSON (JavaScript Object Notation) is the standard format for modern APIs and web configurations. PowerShell provides `ConvertFrom-Json` and `ConvertTo-Json`.

- `ConvertFrom-Json`: Converts JSON string into a custom PowerShell object. *JSON text ko PowerShell object mein badalta hai taaki aap easily usko use kar sako.*
- `ConvertTo-Json`: Converts PowerShell objects back to JSON format, often used when sending data back to an API.

### 3. Interacting with REST APIs

> [!info] Key Concept
> The `Invoke-RestMethod` and `Invoke-WebRequest` cmdlets are used to make HTTP requests (GET, POST, PUT, DELETE) to external web services.

- `Invoke-RestMethod`: The modern and preferred way for APIs. It automatically parses JSON/XML responses into PowerShell objects. *Yeh automatically JSON ko samajh kar objects de deta hai.*
- `Invoke-WebRequest`: Useful if you need to read HTTP headers, status codes, or raw content.

**Authentication:**
APIs usually require authentication. You often pass an API key in the header, or use Basic Auth, or OAuth Bearer tokens.

> [!danger] Common Mistake
> Hardcoding API keys or passwords directly inside the script. Always use secure methods like `Get-Credential`, environment variables, or secret management vaults. *Apne scripts mein passwords plain text mein mat likho.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Administrator access for certain file paths.
> - Internet connection to test API calls.
> - Basic understanding of PowerShell objects.

### Step 1: Read a CSV file and export specific data

Create a dummy CSV file first:
```powershell
# Create dummy data
@"
Name,Department,Location
Amit,IT,Delhi
Rahul,HR,Mumbai
Priya,IT,Pune
"@ | Out-File -FilePath "C:\temp\users.csv"
```

Now, import, filter, and export to a new file:
```powershell
# Import users, filter for IT department, and export
$users = Import-Csv -Path "C:\temp\users.csv"
$itUsers = $users | Where-Object { $_.Department -eq "IT" }
$itUsers | Export-Csv -Path "C:\temp\it_users.csv" -NoTypeInformation
```

> [!success] Expected Output
> ```
> C:\temp\it_users.csv file ban jayegi jismein sirf Amit aur Priya ka data hoga.
> ```

### Step 2: Fetch Data from a Public API

We will use a free public API to fetch JSON data.
```powershell
# Fetch a random joke from an API
$response = Invoke-RestMethod -Uri "https://official-joke-api.appspot.com/random_joke" -Method Get

# Display the data
Write-Host "Setup: $($response.setup)"
Write-Host "Punchline: $($response.punchline)"
```

> [!success] Expected Output
> ```
> Setup: Why did the programmer quit his job?
> Punchline: Because he didn't get arrays.
> ```

### Step 3: Sending Data via API (POST Request)

Often you need to send data, like creating a ticket.
```powershell
# Define the data object
$body = @{
    title = "Server Down"
    body = "The web server is not responding."
    userId = 1
}

# Convert object to JSON
$jsonBody = $body | ConvertTo-Json

# Send POST request
$postResponse = Invoke-RestMethod -Uri "https://jsonplaceholder.typicode.com/posts" -Method Post -Body $jsonBody -ContentType "application/json"

# Check response
$postResponse
```

> [!success] Expected Output
> ```
> id : 101
> title : Server Down
> body : The web server is not responding.
> userId : 1
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-Content` | Text file read karta hai line-by-line | `Get-Content C:\file.txt` |
| `Add-Content` | File mein naya data append karta hai bina overwrite kiye | `Add-Content -Path file.txt -Value "New line"` |
| `Import-Csv` | CSV file ko PowerShell objects mein convert karta hai | `Import-Csv users.csv` |
| `Export-Csv` | Objects ko CSV file mein save karta hai | `$data \| Export-Csv out.csv -NoTypeInformation` |
| `ConvertFrom-Json` | JSON text ko PS object banata hai | `$jsonString \| ConvertFrom-Json` |
| `ConvertTo-Json` | PS object ko JSON string banata hai | `$obj \| ConvertTo-Json -Depth 3` |
| `Invoke-RestMethod` | API call karta hai (GET/POST) aur object return karta hai | `Invoke-RestMethod -Uri "https://api.com"` |
| `Invoke-WebRequest` | Raw HTTP call karta hai (status codes/headers check karne ke liye) | `Invoke-WebRequest -Uri "https://google.com"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `Export-Csv` adds `#TYPE System.Management.Automation.PSCustomObject` | Default behavior of Export-Csv in older PowerShell versions. | Use `-NoTypeInformation` switch. *Hamesha -NoTypeInformation lagaya karo.* |
| API Call returns `401 Unauthorized` | Invalid or missing authentication token in the request headers. | Verify your API token and include it: `-Headers @{"Authorization"="Bearer $token"}` |
| JSON output is cut off or properties are missing | `ConvertTo-Json` defaults to a depth of 2. | Increase the depth parameter: `ConvertTo-Json -Depth 10` |
| `Invoke-RestMethod` returns SSL/TLS error | PowerShell might be using older TLS 1.0 which is blocked by modern APIs. | Force TLS 1.2 at start of script: `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12` |
| Script overwrites the whole log file instead of adding lines | Used `Out-File` or `Set-Content` without append flag. | Use `Add-Content` or `Out-File -Append`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Bulk User Creation from HR Export

> [!example] Ticket
> "HR has provided a CSV file with 50 new joiners. Please create their Active Directory accounts."

**L1 Response:** Verify the CSV file format and ensure all required fields (Name, Department, Manager) are present. *Check karo ki CSV mein koi column missing toh nahi.*
**Escalation Trigger:** If the standard script fails due to unexpected characters or missing OU paths.
**L2 Resolution:** Run the automated PowerShell script utilizing `Import-Csv`. Map the CSV headers to `New-ADUser` parameters and run a loop.

### 🎫 Scenario 2: Service Monitoring to Ticketing API

> [!example] Ticket
> "Need a script to check if a critical service is running, and if not, automatically log a ticket via ServiceNow API."

**L1 Response:** Run the script manually to see if it generates an error. Check network connectivity to ServiceNow API. *Ping ya Test-NetConnection se API server check karo.*
**Escalation Trigger:** The script runs but the API returns HTTP 400 Bad Request.
**L2 Resolution:** Troubleshoot the JSON payload using `ConvertTo-Json`. Ensure mandatory fields required by ServiceNow are included in the POST body via `Invoke-RestMethod`.

### 🎫 Scenario 3: Disk Space Report File Formatting

> [!example] Ticket
> "The automated server disk space script is generating a file, but the output looks like gibberish hash tables instead of a proper table."

**L1 Response:** Open the output file. Check if it's plain text or malformed JSON. *File open karke dekho format kaisa hai.*
**Escalation Trigger:** Unable to fix the formatting in the script.
**L2 Resolution:** The previous admin used `Out-File` on an object array. Fix the script by replacing `Out-File` with `Export-Csv -NoTypeInformation` to create a neat, readable spreadsheet for management.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between `Invoke-RestMethod` and `Invoke-WebRequest`?
> **Answer:** `Invoke-RestMethod` automatically parses JSON or XML responses into PowerShell objects, making it easier to work with API data. `Invoke-WebRequest` returns a raw HTTP web response object, which includes headers, status codes, and raw content string.

==**Exam Tip:** Use RestMethod for processing data, use WebRequest when you need to inspect HTTP Headers or status codes.==

> [!question] Q2: How do you append text to an existing file without overwriting it?
> **Answer:** You use the `Add-Content` cmdlet or use `Out-File -Append`.

==**Exam Tip:** Never use `Set-Content` if your goal is to append, it will wipe the existing data!==

> [!question] Q3: When converting a complex PowerShell object to JSON, some nested properties are missing. How do you fix this?
> **Answer:** By default, `ConvertTo-Json` only serializes 2 levels deep. You need to specify the `-Depth` parameter, like `ConvertTo-Json -Depth 5`.

==**Exam Tip:** The default depth limit is a very common trap in automation scripts interacting with complex APIs.==

> [!question] Q4: Why do we use the `-NoTypeInformation` switch with `Export-Csv`?
> **Answer:** Without it, older versions of PowerShell (v5.1 and below) will insert a `#TYPE` header line at the very top of the CSV file, which can break other tools (like Excel or databases) trying to read the CSV file.

==**Exam Tip:** In PowerShell 7+, `-NoTypeInformation` is the default behavior.==

> [!question] Q5: How do you pass an authentication token to a REST API in PowerShell?
> **Answer:** You pass it via the `-Headers` parameter in `Invoke-RestMethod`. Usually as a hash table: `@{"Authorization" = "Bearer $token"}`.

---
## 🔗 Related Notes

- [[PS-01 Intro to PowerShell|PowerShell Basics]] — Core commands and objects.
- [[PS-02 Variables and Loops|Loops and Logic]] — Needed to iterate through CSV data.
- [[AD-04 Bulk Automation|Active Directory Bulk Tasks]] — Real-world example of using CSV files.
- [[Cloud-02 Introduction to REST APIs|API Basics]] — General concepts of how APIs work.
