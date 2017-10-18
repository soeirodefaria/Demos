#GCP - Storage/Datastore/PubSub image resizing app demo

This is an introductory demo that shows how to leverage GCP services and cloud best practices (decoupling, high availability, 
Disaster Recovery, Scalability) to create a web application.

This demo has a SOA design, using PHP for the frontend and Python in the backend. The demo web app allows users to fill out
a short form with Name, Email, and Picture. The picture is resized, and another page shows the resized picture of all users.
The idea is to simulate a social media app, but the concepts can be applied to other examples.

The app has 3 main services:

Front End: hosting the main page, and ingesting the form from users
Back End: transforming the picture into thumbnail size
Render: displaying all users' thumbnail pictures

Intended app setup, logic, and flow:

- Global Load Balancer 
  - 3 backends
    - frontend
    - render
    - bucket with thumbnail pictures (CDN enabled)

- Multi Regional GCS bucket

- Front End
  - 1 or more Instance Groups (in different regions)
  - PHP + GCP APIs (Storage and PubSub)
  - Allows users to upload picture
  - Upload picture into a temp folder in GCS bucket
  - Sends a message to a PubSub topic
    - message content: GCS object location, name, email

- Back End:
  - 1 or more Instance Groups (in different regions)
  - Python + GCP APIs (Storage, PubSub, Datastore)
  - Subscribe to the PubSub topic, reads the message
  - download temp picture, resizes, uploads to GCS bucket
  - create an entry in Datastore:
    - email (as the index); name; thumbnail url; data uploaded (important for rendering later)
  - if everything suceeds .... acknoledge the message (removing from the PubSub topic, to avoid message to be processed again)

- Render:
  - 1 or more Instance Groups (in different regions)
  - PHP + GCP API (Datastore)
  - Queries Data store, sort entities by the newest (so we can display the newest picture 1st)
  - creates an array in PHP, loop, and display
  
  
Highlights/Teachble points:

- GCP Global Networking, LB is Global, one single IP (*High Availability*)
- Due to global networking, we can have VMs running our services in multiple regions (*High Availability* / *DR* / *Scalability*)
- PubSub (*Decoupling*)
- Datastore (*Decoupling* / *Scalability*)
- Storage (*High Availability* / *DR*)
- GCE with Autoscaling, Templates (*Scalability* / *Repeatable*)



NOTE: I am not and do not claim to be a web developer, or a developer at all. The demo serves its purpose of proving a few points.
Feedback is more than welcome, but do it in a nicer and non-derrogatory way. I'd love to incorporate feedback and have a more 
robust and elegant looking app.

