# Solidity Car Rental Contract Documentation  

## Summary of the Contract 

  

The contract is a decentralized car rental platform. It has two main entities: `User` and `Car`. Users can rent cars, make payments, and manage their accounts. Cars can be added, edited, and their statuses can be changed by the owner of the contract. The contract also keeps track of the total payments made to it. 

  

### Here's a breakdown of the main functionalities: 

  

1. `Ownership`: The contract has an owner who can change the ownership, add cars, edit car metadata, and withdraw the balance. 

   

2. `Users`: Users can be added to the platform. They can deposit funds, rent cars, return cars, and make payments. Each user has a balance and may accumulate debt based on the time they've rented a car. 

  

3. `Cars`: Cars have metadata like name, image URL, rent fee, and sale fee. They also have a status that can be "Retired," "In Use," or "Available." 

 4. `Renting`: Users can "check out" a car, which changes the car's status to "In Use" and starts a timer. When they "check in" the car, the timer stops, and the user's debt is calculated based on the time and rent fee. 

5. `Payments`: Users can deposit funds into their account, make payments to clear their debt, and withdraw their balance. The owner can also withdraw the total payments made to the contract. 

  

6. `Events`: Various events are emitted for different actions like adding a car, editing car metadata, user actions, etc. 

  

7. `Modifiers and Checks`: The contract uses various modifiers and require statements to enforce rules like "only the owner can add a car," "users must exist to rent a car," etc. 

  

8. `ReentrancyGuard`: The contract uses **OpenZeppelin's ReentrancyGuard** to prevent re-entrant attacks, particularly in the withdrawal functions. 

  

9. `Counters`: OpenZeppelin's Counters library is used for auto-incrementing the car IDs. 

 

## Car Rental Contract 

 ```solidity

// SPDX-License-Identifier: MIT 

// Specify the compiler version and import necessary libraries 

pragma solidity ^0.8.7; 

import "@openzeppelin/contracts/utils/Counters.sol"; 

import "@openzeppelin/contracts/security/ReentrancyGuard.sol"; 

 
 

// Main contract 

contract RentalPlatform is ReentrancyGuard { 

  // Use OpenZeppelin's Counters library for auto-incrementing IDs 

  using Counters for Counters.Counter; 

  Counters.Counter private _counter; 

 
 

  // DATA 

   

  // Address of the contract owner 

  address private owner; 

 
 

  // Total payments made to the contract 

  uint private totalPayments; 

 
 

  // User struct to hold user information 

  struct User { 

    address walletAddress; 

    string name; 

    string lastname; 

    uint rentedCarId; 

    uint balance; 

    uint debt; 

    uint start; 

    uint end; 

  } 

 
 

  // Car struct to hold car information 

  struct Car { 

    uint id; 

    string name; 

    string imgUrl; 

    Status status; 

    uint rentFee; 

    uint saleFee; 

  } 

 
 

  // Enum to indicate the status of a car 

  enum Status { Retired, InUse, Available } 

 
 

  // EVENTS 

   

  // Events to log various actions 

  event CarAdded(uint indexed id, string name, string imgUrl, uint rentFee, uint saleFee); 

  event CarMetadataEdited(uint indexed id, string name, string imgUrl, uint rentFee, uint saleFee); 

  event CarStatusEdited(uint indexed id, Status status); 

  event UserAdded(address indexed walletAddress, string name, string lastname); 

  event Deposit(address indexed walletAddress, uint amount); 

  event CheckOut(address indexed walletAddress, uint indexed carId); 

  event CheckIn(address indexed walletAddress, uint indexed carId); 

  event PaymentMade(address indexed walletAddress, uint amount); 

  event BalanceWithdrawn(address indexed walletAddress, uint amount); 

 
 

  // MAPPINGS 

   

  // Mapping to store user data 

  mapping(address => User) private users; 

 
 

  // Mapping to store car data 

  mapping(uint => Car) private cars; 

 
 

  // CONSTRUCTOR 

   

  // Initialize contract with the deployer as the owner 

  constructor() { 

    owner = msg.sender; 

    totalPayments = 0; 

  } 

 
 

  // MODIFIERS 

   

  // Modifier to restrict function access to only the owner 

  modifier onlyOwner() { 

    require(msg.sender == owner, "Only the owner can call this function"); 

    _; 

  } 

 
 

  // FUNCTIONS 

   

  // EXECUTE FUNCTIONS 

   

  // Change the owner of the contract 

  function setOwner(address _newOwner) external onlyOwner { 

    owner = _newOwner; 

  } 

 
 

  // Add a new user to the platform 

  function addUser(string calldata name, string calldata lastname) external { 

    require(!isUser(msg.sender), "User already exists!"); 

    users[msg.sender] = User(msg.sender, name, lastname, 0, 0, 0, 0, 0); 

    emit UserAdded(msg.sender, users[msg.sender].name, 	users[msg.sender].lastname); 

  } 

 
 

  // Add a new car to the platform 

  function addCar(string calldata name, string calldata url, uint rent, uint sale) external onlyOwner { 

    _counter.increment(); 

    uint counter = _counter.current(); 

    cars[counter] = Car(counter, name, url, Status.Available, rent, sale); 

    emit CarAdded(counter, cars[counter].name, cars[counter].imgUrl, cars[counter].rentFee, cars[counter].saleFee); 

  } 

 
 

  // Edit metadata of an existing car 

  function editCarMetadata(uint id, string calldata name, string calldata imgUrl, uint rentFee, uint saleFee) external onlyOwner { 

    require(cars[id].id != 0, "Car with given ID does not exist."); 

    Car storage car = cars[id]; 

    if(bytes(name).length != 0) { 

        car.name = name; 

    } 

    if(bytes(imgUrl).length != 0) { 

        car.imgUrl = imgUrl; 

    } 

    if(rentFee > 0) { 

        car.rentFee = rentFee; 

    } 

    if(saleFee > 0) { 

        car.saleFee = saleFee; 

    } 

    emit CarMetadataEdited(id, car.name, car.imgUrl, car.rentFee, car.saleFee); 

  } 

 
 

  // Edit the status of an existing car 

  function editCarStatus(uint id, Status status) external onlyOwner { 

    require(cars[id].id != 0, "Car with given ID does not exist."); 

    cars[id].status = status; 

    emit CarStatusEdited(id, status); 

  } 

 
 

  // Rent a car 

  function checkOut(uint id) external { 

    require(isUser(msg.sender), "User does not exist!"); 

    require(cars[id].status == Status.Available, "Car is not available for rent!"); 

    require(users[msg.sender].rentedCarId == 0, "User has already rented a car!"); 

    require(users[msg.sender].debt == 0, "User has an outstanding debt!"); 

 
 

    users[msg.sender].start = block.timestamp; 

    users[msg.sender].rentedCarId = id; 

    cars[id].status =  Status.InUse; 

    emit CheckOut(msg.sender, id); 

  } 

 
 

  // Return a rented car 

  function checkIn() external { 

    require(isUser(msg.sender), "User does not exist!"); 

    uint rentedCarId = users[msg.sender].rentedCarId; 

    require(rentedCarId != 0, "User has not rented a car!"); 

 
 

    uint usedSeconds = block.timestamp - users[msg.sender].start; 

    uint rentFee = cars[rentedCarId].rentFee; 

    users[msg.sender].debt += calculateDebt(usedSeconds, rentFee); 

 
 

    users[msg.sender].rentedCarId = 0; 

    users[msg.sender].start = 0; 

    users[msg.sender].end = 0; 

    cars[rentedCarId].status = Status.Available; 

    emit CheckIn(msg.sender, rentedCarId); 

  } 

 
 

  // Deposit funds into user's account 

  function deposit() external payable { 

    require(isUser(msg.sender), "User does not exist!"); 

    users[msg.sender].balance += msg.value; 

    emit Deposit(msg.sender, msg.value); 

  } 

 
 

  // Make payment to clear debt 

  function makePayment() external { 

    require(isUser(msg.sender), "User does not exist!"); 

    uint debt = users[msg.sender].debt; 

    uint balance = users[msg.sender].balance; 

    require(debt > 0, "User has no debt!"); 

    require(balance >= debt, "User has insufficient balance!"); 

 
 

    unchecked { 

      users[msg.sender].balance -= debt; 

    } 

    totalPayments += debt; 

    users[msg.sender].debt = 0; 

    emit PaymentMade(msg.sender, debt); 

  } 

 
 

  // Withdraw balance from user's account 

  function withdrawBalance(uint amount) external nonReentrant { 

    require(isUser(msg.sender), "User does not exist!"); 

    uint balance = users[msg.sender].balance; 

    require(balance >= amount, "Insufficient balance to withdraw!"); 

 
 

    users[msg.sender].balance -= amount; 

    (bool success, ) = msg.sender.call{value: amount}(""); 

    require(success, "Transfer failed."); 

    emit BalanceWithdrawn(msg.sender, amount); 

  } 

 
 

  // Withdraw balance from the contract (only owner) 

  function withdrawOwnerBalance(uint amount) external onlyOwner { 

    require(totalPayments >= amount, "Insufficient contract balance to withdraw!"); 

 
 

    (bool success, ) = owner.call{value: amount}(""); 

    require(success, "Transfer failed."); 

    totalPayments -= amount; 

    emit BalanceWithdrawn(owner, amount); 

  } 

 
 

  // QUERY FUNCTIONS 

   

  // Get the owner of the contract 

  function getOwner() external view returns(address) { 

    return owner; 

  } 

 
 

  // Check if an address is a user 

  function isUser(address walletAddress) internal view returns(bool) { 

    return users[walletAddress].walletAddress != address(0); 

  } 

 
 

  // Get user details 

  function getUser(address walletAddress) external view returns (User memory) { 

    require(isUser(walletAddress), "User does not exist!"); 

    return users[walletAddress]; 

  } 

 
 

  // Get car details 

  function getCar(uint id) external view returns(Car memory) { 

    require(cars[id].id != 0, "Car does not exist!"); 

    return cars[id]; 

  } 

 
 

  // Get cars by their status 

  function getCarsByStatus(Status _status) external view returns (Car[] memory) { 

    uint count = 0; 

    uint length = _counter.current(); 

    for (uint i = 1; i <= length; i++) { 

        if (cars[i].status == _status) { 

            count++; 

        } 

    } 

    Car[] memory carsWithStatus = new Car[](count); 

    count = 0; 

    for (uint i = 1; i <= length; i++) { 

      if (cars[i].status == _status) { 

        carsWithStatus[count] = cars[i]; 

        count++; 

      } 

    } 

    return carsWithStatus; 

} 

 
 

  // Calculate debt based on time and rent fee 

  function calculateDebt(uint usedSeconds, uint rentFee) private pure returns (uint) { 

    uint usedMinutes = usedSeconds / 60; 

    return usedMinutes * rentFee; 

  } 

 
 

  // Get the current count of cars 

  function getCurrentCount() external view returns (uint) { 

    return _counter.current(); 

  } 

 
 

  // Get the balance of the contract (only owner) 

  function getContractBalance() external view onlyOwner returns (uint) { 

    return address(this).balance; 

  } 

 
 

  // Get the total payments made to the contract (only owner) 

  function getTotalPayments() external view onlyOwner returns (uint) { 

    return totalPayments; 

  } 

 
} 

```

## Overview 

This document serves as a comprehensive guide for frontend developers and testers to understand how to interact with the Solidity Car Rental Contract. The contract is designed to manage a decentralized car rental platform. 

### Key Entities 

`User`: Represents a user on the platform. 

`Car`: Represents a car available for rent. 

### Key Functions 

- Owner-related functions 

- User-related functions 

- Car-related functions 

- Payment and balance functions 

 

### Functions 

#### Owner-Related Functions 

`setOwner(address _newOwner) `

- Access: Only by the current owner. 

- Parameters: _newOwner - Address of the new owner. 

- Purpose: Changes the ownership of the contract. 

### User-Related Functions 

`addUser(string name, string lastname)` 

- Access: Anyone who is not already a user. 

- Parameters: name and lastname - Strings representing the user's first and last name. 

- Purpose: Adds a new user to the platform. 

`deposit() `

- Access: Existing users. 

- Parameters: None. The function is payable. 

- Purpose: Deposits funds into the user's account. 

`makePayment()` 

- Access: Existing users with a debt. 

- Parameters: None. 

- Purpose: Clears the user's debt. 

`withdrawBalance(uint amount)` 

- Access: Existing users. 

- Parameters: amount - The amount to withdraw. 

- Purpose: Withdraws the specified amount from the user's balance. 

### Car-Related Functions 

`addCar(string name, string url, uint rent, uint sale)` 

- Access: Only by the owner. 

- Parameters: name, url, rent, sale - Car details. 

- Purpose: Adds a new car to the platform. 

`editCarMetadata(uint id, string name, string imgUrl, uint rentFee, uint saleFee) `

- Access: Only by the owner. 

- Parameters: Car ID and new details. 

- Purpose: Edits the metadata of an existing car. 

`editCarStatus(uint id, Status status)` 

- Access: Only by the owner. 

- Parameters: id - Car ID, status - New status. 

- Purpose: Changes the status of a car. 

`checkOut(uint id)` 

- Access: Existing users who have no debt and have not rented a car. 

- Parameters: id - Car ID. 

- Purpose: Rents a car. 

`checkIn()` 

- Access: Users who have rented a car. 

- Parameters: None. 

- Purpose: Returns the rented car and calculates the debt. 

### Query Functions 

`getOwner()` 

- Returns: Address of the current owner. 

`getUser(address walletAddress)` 

- Parameters: walletAddress - Address of the user. 

- Returns: User details. 

`getCar(uint id)` 

- Parameters: id - Car ID. 

- Returns: Car details. 

`getCarsByStatus(Status _status)` 

- Parameters: _status - Status to filter by. 

- Returns: Array of cars with the given status. 

`getCurrentCount()` 

- Returns: Current count of cars. 

`getContractBalance()` 

- Access: Only by the owner. 

- Returns: Contract's balance. 

`getTotalPayments()` 

- Access: Only by the owner. 

- Returns: Total payments made to the contract. 
