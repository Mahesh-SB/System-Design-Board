# Uber

## HLD

![image](https://github.com/user-attachments/assets/d5885eb9-7ffe-4beb-86f8-bb65f0b05227)

## Class, interface and Enumerations
## Class : 
1. User.
2. CabDetail.
3. RiderDetail
4. RideDetail
5. PaymentDetail
6. Invoice

## Enumerations :
1. CabType
2. RideType
3. PaymentType

## API :
1. GET    /users/{userId}
2. GET    /users/{userId}/rides
3. GET    /users/{userId}/rides/{rideId}
4. GET    /drivers/{driverId}/rides
5. GET    /drivers/{driverId}/rides/{rideId}
6. GET    /rides/fare-estimate?pickup=lat,lng&drop=lat,lng
7. GET    /rides/{rideId}/route?start=lat,lng&end=lat,lng
8. POST   /payments
9. POST   /drivers/{driverId}/location
10. POST   /rides
11. POST   /rides/{rideId}/start
12. POST   /rides/{rideId}/cancel
13. POST   /rides/{rideId}/end

    
