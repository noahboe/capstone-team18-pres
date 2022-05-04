---
marp: true
title: Team 18 Full Presentation
---

<h1></h1>
<h1 align="center"> Library Database Project </h1>
<h2 align="center"> Capstone Team 18 </h2>
<h3 align="center"> Team Members: </h3>
<p align="center"> Noah Boeckman, Christopher Chhim, Bach Ngo, Kyle Weidner </p>

---

# Unit Testing the cyberCommons Framework
Chris Chimm, Noah Boeckman, Bach Ngo, Kyle Weidner

---

# The cyberCommons Framework​
- An architecture for distributed computing workflows​
- A Python REST API built with Django to manage:​
  - Workers built with Celery, an asynchronous task queue​
  - Communication through RabbitMQ, a message broker​
  - A MongoDB document based database​

---

# The cyberCommons Framework
![bg right width:600](https://cybercom-docs.readthedocs.io/en/latest/_images/cybercommons.png)

---

# The cyberCommons Framework
- Where its used:
  - University of Colorado Libraries ​
  - University of Oklahoma Libraries ​
  - US Congressional Hearings Search Engine ​
  - Latin Search Engine ​
  - Northern Arizona University EcoPAD ​
  - The Oklahoma Biological Survey ​
  - The Earth Observation Modeling facility ​
  - The South Central Climate Sciences Center ​
  - The Oklahoma Water Survey​

---

# Unit Testing goals
- Ensure that all parts of the API are avaliable 
- Verify that authentication is correct
- Ensure current functional parts behave correctly

---

# Ensuring availability
- Making get requests as a simulated web browser
  - Using Djangos built-in ```APIClient``` 
```python
request = self.client.get('/catalog/data/')
```
- Testing that the API returns the correct http status code
```python
self.assertEqual(request.status_code, 200)
```

---

# Verifying authentication
- Creating different users with Djangos authentication models
```python
self.user = User.objects.create_user(
            username='test_user',
        )
self.super_user = User.objects.create_superuser(
            username='test_super_user',
        )
```

---

# Verifying authentication
- Making sure users not logged in get a 401 "unauthorized" response
```python
request = self.client.get('/user/')
self.assertEqual(request.status_code, 401)
```
- Authenticating the simulated client before the request to get a 200 "OK" response
```python
self.client.force_authenticate(user=self.user)
request = self.client.get('/user/')
self.assertEqual(request.status_code, 200)
```

---

# Ensuring functionality 
- Testing the email function of a worker by sending a post request as a logged in user
```python
self.client.force_authenticate(user=self.user)
request = self.client.post('/queue/run/emailq.tasks.tasks.sendmail/', 
        {'args':['fakeemail@nowhere.com', 'subject', 'body']}, format = "json")
```
- Checking that the worker returns a result url that is accessable
```python
url = str(request.data.get('result_url')[21:])
response = self.client.get(url)
self.assertEqual(response.status_code, 200)
```

---

# Ensuring functionality
- Testing that a logged in admin can view missing meta data from a worker
```python
self.client.force_authenticate( user=self.user)
request = self.client.post( '/queue/run/dspaceq.tasks.tasks.list_missing_metadata_etd/', 
        {'queue': 'shareok-dspace6x-test-workerq','args': []}, format = "json")
```
- Verifying results from the response url
```python
url = str(request.data.get('result_url')[21:])
response = self.client.get(url)
self.assertEqual(response.status_code, 200)
self.assertEqual(response.data['result']['status'], 'SUCCESS')
```

---