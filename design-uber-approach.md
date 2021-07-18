# Design Uber System

## Introduction
* Uber is primarily a ride hailing service. It is a platform to connect riders to drivers so that the riders can reach their destination. 
* Some of other platforms providing similar services are Ola, Lyft, Gojek, Via, Grab and Bolt.

## Requirement analysis 
### Functional requirements

* Both drivers and riders should be able to continuously send out their current location
* Riders should be able to see the nearby cabs
* Riders should be able to book a cab. They should also be able to cancel a cab after booking but before boarding it.
* Drivers should be able to accept or reject the request for bookings.
### Non-functional requirements

* The system should be reliable. It should always be up and running.
* The system should have high performance.
### Additional requirements

* Our system should be able to suggest the shortest path for the trip.
* The trip should be completed after arriving at the destination.
* Our system should be able to accept payments.
* We should be able to save the trip information.
* We should be able to restrict the number of cancellations a user can make on a day.
* Our system should be able to implement surge pricing.
* The system should be consistent. A rider by mistake should not be able to do multiple bookings at the same time. 
* Similarly the driver should not be able accept multiple bookings. 
* Such scenarios should be taken care.
* Preferred Access points. The system should prompt for a preferred pickup location if it determines that it cannot come to the rider’s exact location.
* Our system should have the telemetry data to perform analytics.
* The drivers and riders should have ratings associated with them.
## API design
### Rider to system apis :

* updateLocation(currentLocation)
* searchCabs(userId, destination)
* bookRide(userId, carType, destination)
* cancelRide(userId)
* Driver to system apis :

* updateLocation(currentLocation)
* acceptRide(userId)
* rejectRide(userId)
* cancelRide(userId)
* completeTrip(userId)
### System to rider apis :

* showCabs(Map<CarType, Price> priceList)
* bookingAcceptedNotification(DriverDetails, CarDetails)
* rideCancelled()
* retryAgainNotification(Message)

### System to driver apis :

* rideRequest()
* rideCancelledNotification()

## Define data model
* We will not be defining the data model as we have clearly defined it as addition requirement to save the data.

## Capacity estimations
* Consider we have 10 Million bookings per day. Each booking will generate at an average of 2MB of data.

* So per day we will generate
10 Million * 2 MB = 20 Million MB = 20,000,000 Mb = 20,000 GB = 20 TB of data.

## High level design
* Let us see a simple design for the given problem statement.

* The driver updates his location to the supply service.
* The supply service forwards the location data to the location service.
* The rider opens his app, sends his location and requests for a cab by sending a request to Demand service.
* The demand service sends the request to the location service.
* The location service responds with the nearby cab details.
* The supply service sends the cab details to the demand service.
* The demand service sends the cab details to the rider.

#### What are the challenges we can see?

* The driver and rider have to update their current location continuously. We can use websockets for data transfer.
* We are bombarding our demand and supply service continuously with the location data say every second. * We can introduce a queue like Kafka between the riders or drivers and demand supply services.
### How to maintain the location data?

* We draw a circle around our GPS location of radius say 1 KM and book the cab which is present in the 1 KM radius. It is a good but computationally costly solution
* Create grids which partition the area into several blocks.
*  Which shape should we use to create the grid so that it covers the entire earth. We have choices of only 3 shapes triangle, square and hexagon.
* Existing libraries to maintain location data?

* Google’s S2 location library – Uses squares to create grid.
* Uber’s H3 library – Uses hexagons to create grid.

### Steps in updated design :

* The driver creates a WebSocket session and updates its location.
* The WebSocket service puts the location in the Kafka queue.
* This location is read by the supply service.
* The supply service sends this location to the location service. The location service maps the driver to the respective hexagon according to the location.
* The rider creates a WebSocket session, updates its location and request for a ride.
* The WebSocket service puts the location in the Kafka queue.
* The demand service reads the location from the Kafka queue.
* The demand service sends the data to location service.
* The location service maps the rider to the hexagon and finds the drivers in the neighboring hexagon and sends their data to the demand service.
* The demand service fixes a rider and sends the information to the notification service.
* The notification service sends a rideRequest to the driver.
* The driver chooses to accept or reject.
  *  A. If the driver rejects then the notification service informs this to demand service which finds a different driver.
  * B. If the driver accepts then the notification service notifies the rider along with all the driver and car details. Parallelly the Driver gets the rider details.
* A. Suppose the driver chooses to cancel the ride after accepting it sends the message to notification service.
* B. The notification service informs the rider about the cancellation and sends a message saying that a new driver will be allocated.
* C. The notification service parallelly informs the Demand service about the cancellation so that the demand service chooses another driver for the trip.


## Scale the design
* We have 2 non-functional requirements.

* 1. The system should be reliable. It should always be up and running – This can be accomplished using the no single point of failure principle. Here we add redundant copies of each of the components.

* 2. The system should have high performance – This can be accomplished using the no bottleneck principle. We have to use Kafka queue and consistent hashing to divide the traffic.

#### Scaling location service?

* RingPop design with SWIM protocol.

## Additional things
### Additional requirements
* Our system should be able to suggest the shortest path for the trip – Use Google maps API libraries which internally use Dijkstra’s Algorithm to find the shortest path.
* The trip should be completed after arriving at the destination – The driver will call an api informing the trip is completed.
* Our system should be able to accept payments – Have a payments service.
* We should be able to save the trip information – Use NoSQL database.
* We should be able to restrict the number of cancellations a user can make on a day – Use a rate limiter service.
* Our system should be able to implement surge pricing – We can create heatmaps based on the demand of cabs in a particular area.
* The system should be consistent – We need to use distributed locking to provide consistency. Reference link.
* Preferred Access points. The system should prompt for a preferred pickup location if it determines that it cannot come to the rider’s exact location –  This can be based on the trend of previous riders bookings.
* Our system should have the telemetry data to perform analytics – There are various data analytics tools like Hadoop, Spark which can be leveraged for this purpose.
* The drivers and riders should have ratings associated with them – Have a recommendation system.

