# LLD-Smart-Parking-Lot-System

## Problem Statement:
Imagine a parking lot in an urban area with multiple floors and numerous parking spots. Your task is to create a low-level design for a system that efficiently manages the parking process. The system should automatically assign parking spots based on vehicle size and availability, track the time each vehicle spends in the parking lot, and calculate parking fees upon exit.

## Functional Requirements:
- **Parking Spot Allocation**: Automatically assign an available parking spot to a vehicle when it enters, based on the vehicle’s size (e.g., motorcycle, car, bus).
- **Check-In and Check-Out**: Record the entry and exit times of vehicles.
- **Parking Fee Calculation**: Calculate fees based on the duration of stay and vehicle type.
- **Real-Time Availability Update**: Update the availability of parking spots in real-time as vehicles enter and leave.

## Design Aspects to Consider:
- **Data Model**: Design a database schema to manage parking spots, vehicles, and parking transactions.

- **Algorithm for Spot Allocation**: Develop an algorithm to efficiently assign parking spots to incoming vehicles.

- **Fee Calculation Logic**: Implement logic to calculate fees based on parking duration and vehicle type.

- **Concurrency Handling**: Ensure the system can handle multiple vehicles entering or exiting simultaneously.


## **1. Data Model**

### List of tables and entities

1. ParkingLots
2. Floors
3. ParkingSpots
4. Vehicles
5. ParkingTransactions

#### ParkingLots

CREATE TABLE ParkingLots (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    address VARCHAR(255)
);


#### Floors

CREATE TABLE Floors (
    id INT PRIMARY KEY,
    lotId INT,
    FOREIGN KEY (lotId) REFERENCES ParkingLots(id)
);


#### ParkingSpots

CREATE TABLE ParkingSpots (
    id INT PRIMARY KEY,
    floorId INT,
    type VARCHAR(50),
    occupied BOOLEAN,
    FOREIGN KEY (floorId) REFERENCES Floors(id)
);


#### Vehicles

CREATE TABLE Vehicles (
    id INT PRIMARY KEY,
    type VARCHAR(50),
    size VARCHAR(50)
);


#### ParkingTransactions

CREATE TABLE ParkingTransactions (
    id INT PRIMARY KEY,
    vehicleId INT,
    spotId INT,
    entryTime DATETIME,
    exitTime DATETIME,
    fee FLOAT,
    FOREIGN KEY (vehicleId) REFERENCES Vehicles(id),
    FOREIGN KEY (spotId) REFERENCES ParkingSpots(id)
);


## **2. Algorithm for Spot Allocation and Fee Calculation** ##
- **ParkingLot:**
    - Manages the entire parking lot, which includes multiple floors.
    - Handles vehicle check-in and check-out processes.
    - Maintains a list of all parking transactions.

- **Floor:**
    - Represents a single floor in the parking lot.
    - Manages a collection of parking spots on that floor.
    - Provides functionality to find available spots based on vehicle type.

- **ParkingSpot:**
    - Represents an individual parking spot.
    - Tracks its own type (e.g., motorcycle, car, bus) and occupancy status.
    - Provides methods to assign and release a vehicle.

- **ParkingTransaction:**
    - Represents a parking transaction for a vehicle.
    - Records the vehicle's entry and exit times and calculates the parking fee based on the duration of stay.

class ParkingLot {
    constructor(floors) {
        this.floors = floors;
        this.transactions = [];
    }

    findSpotForVehicle(vehicle) {
        for (let floor of this.floors) {
            let spot = floor.getAvailableSpot(vehicle.type);
            if (spot) return spot;
        }
        return null;
    }

    checkIn(vehicle) {
        let spot = this.findSpotForVehicle(vehicle);
        if (spot) {
            spot.assignVehicle(vehicle);
            let transaction = new ParkingTransaction(vehicle.id, spot.id, new Date());
            this.transactions.push(transaction);
            return transaction;
        } else {
            throw new Error("No available spot for this vehicle");
        }
    }

    checkOut(transactionId) {
        let transaction = this.transactions.find(t => t.id === transactionId);
        if (transaction) {
            transaction.exitTime = new Date();
            transaction.fee = transaction.calculateFee();
            let spot = this.floors.flatMap(floor => floor.spots).find(s => s.id === transaction.spotId);
            spot.releaseSpot();
            return transaction.fee;
        } else {
            throw new Error("Invalid transaction ID");
        }
    }
}

class Floor {
    constructor(id, spots) {
        this.id = id;
        this.spots = spots;
    }

    getAvailableSpot(vehicleType) {
        return this.spots.find(spot => !spot.occupied && spot.type === vehicleType);
    }
}

class ParkingSpot {
    constructor(id, type) {
        this.id = id;
        this.type = type;
        this.occupied = false;
    }

    assignVehicle(vehicle) {
        this.occupied = true;
    }

    releaseSpot() {
        this.occupied = false;
    }
}

class ParkingTransaction {
    constructor(vehicleId, spotId, entryTime) {
        this.id = Math.floor(Math.random() * 1000000);
        this.vehicleId = vehicleId;
        this.spotId = spotId;
        this.entryTime = entryTime;
        this.exitTime = null;
        this.fee = 0;
    }

    calculateFee() {
        const rate = 10; // Example rate per hour
        let duration = (this.exitTime - this.entryTime) / (1000 * 60 * 60); // duration in hours
        return Math.ceil(duration) * rate;
    }
}


## **4. Concurrency Handling**
In a high-traffic parking lot, multiple vehicles can enter or exit simultaneously. Our system needs to handle these concurrent operations efficiently and accurately to avoid data inconsistencies.

- **Database Transactions:** Use database transactions to ensure that a series of operations are executed as a single unit. If any operation fails, the entire transaction is rolled back Transactions can prevent partial updates and maintain data consistency.

- **Row Locking:** Implement row-level locking to prevent multiple processes from accessing and modifying the same row concurrently. For example, when updating a parking spot's occupancy status, the row can be locked until the operation is complete.

- **Optimistic Concurrency Control:** Use versioning or timestamps to detect conflicts. If a conflict is detected (i.e., if another transaction modified the data), the transaction can be retried.

- **Atomic Operations:** Ensure that critical operations (e.g., finding and assigning a spot) are atomic. This can be achieved by combining the read and write operations into a single database query.
