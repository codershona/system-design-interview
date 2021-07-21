# Design movie ticket booking system


## Introduction
* There are many online portals where you can book tickets for movies and live events. Some famous examples are bookmyshow and ticketmaster.

## Requirement analysis
### Functional requirements
* The user should be able to see the list of cities -> choose a city -> see the list of theatres in that city -> choose a theatre -> see the shows running in that theatre -> choose a show -> see the seating layout along with already booked and available seats -> reserve a seat -> make payment within stipulated time -> the seat is booked and we receive confirmation via email or sms.

* We can also view the ratings and comments for the movie.

### Non-functional requirements
* The system should always be available
* The system should be responsive. The user experience should be smooth.

## API design
* List<City> getCities()
* List<Theater> getTheaters(city)
* List<Show> getShows(theater)
* List<List<Seats>> getSeats(show)
* Bill researveSeat(seats)
* Invoice makePayment(bill)

## Define the data model
* We primarily have 2 types of data here

* Structured relational data – We need to have tables to save the City, Theatre, Show and seating arrangement.
Unstructured data – Comments, ratings and movie descriptions are best saved in some NoSQL database.
<br/>

## Back-of-the-envelope calculations
* Let us consider that the interviewer did not ask for any calculations. So we do not need to perform any calculations.

* As an exercise consider you are the interviewer. What calculation would you ask?

## High level design
* We have 2 approaches to design the system.

  * First approach is that let our system handle everything. In this case we are in charge of the bookings.
  * The second approach is that we are only a mediator.
<br/>

* A simple flow using the first diagram.

  * The user sends request to book a ticket for a PVR movie to our app server.
  * Our app server calls the apis of PVR cinemas and requests them to reserve the tickets. This is done so that this seat is locked for the user for say 10 minutes and no other user can reserve it.
  * Our app server gets a success response from the PVR cinemas and it redirects the user to the payment page.
  * The user makes the payment with the help of the app server.
  * Our app server notifies to the PVR cinemas that payment is made.
  * The PVR cinemas marks the seat as booked and returns a success response to our app server.
  * Our app server notifies the user that the booking is confirmed.
  * In case the payment is made and before our app server notifies the PVR cinemas the 10 minute window expires and someone else reserves the seat then we will inform this to our user and refund the amount.

* In the first diagram our app server is doing too many things. It is responsible for calling the apis of PVR or Inox, it is also responsible to handle the payments and notify the users. Let us use single responsibility principle and break down our app server.

#### Architectural pattern to use?

* A mix of SOA and microservices.

* This microservice based system has many advantages.

    * One is that if PVR system has some issue it will affect only PVR microservice and not others.
    * Failure of one microservice will have no effect on other microservices.
    * The system is extensible. In future suppose we want to handle live events we can follow the same approach of reusing or creating a new microservice template.




## Scaling the design
* Let us revisit our non-functional requirements

* The system should always be available, we can use no single point of failure principle and create replicas so in the event of failure of one component other can take over. This will much easier since we have microservices which can restart themselves or create new instances in the event of failures.
* The system should be responsive. The user experience should be smooth. We have a queue and a notification service which runs asynchronously from the main process to improve the performance. We have also added caching for performance reasons.
