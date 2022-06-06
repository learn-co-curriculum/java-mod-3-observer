# Observer

## Learning Goals

- Explain the observer pattern
- Use the pattern in Java

## Introduction

Here's when you would want to use the pattern:

- Use to model an object hierarchy where changes to one object, called the
  subject or the observed, should have an impact on other objects, called the
  observers, but you don't know ahead of time who these observers are.
- You also don't want to make assumptions about the observers, i.e. you don't
  want to have your subject tightly coupled with your observers.

Let's add a `checkIn()` method to our `HotelRoom` class. Note that we don't add
it to our `HotelRoomInterface` because this action does not apply to floors -
each room has to be checked in individually and by a different person:

```java
class HotelRoom implements HotelRoomInterface {
    public void book(String guestName) {
        Logger.getInstance().log("Booked a room for " + guestName);
    }

    public void clean() {
        Logger.getInstance().log("Cleaned room");
    }

    public void checkIn(String guestName) {
        Logger.getInstance().log(guestName + "checked in");
    }
}
```

When a guest checks in for a specific room, there are actions that the room must
take, which we would implement in the `checkIn()` method of the `HotelRoom`
class, where that functionality belongs. We may, for example, want to add the
room to the cleaning schedule so that the cleaning staff knows to include it in
their daily service.

There may, however, be other functionality that should be triggered when a room
is booked, but that the room class itself either shouldn't be responsible for
implementing or couldn't be responsible for implementing it because that
functionality didn't exist when the `HotelRoom` class was first created. Let's
say, for example, that we want to send a welcome email and grant loyalty points
to a guest when they check in.

- welcome email
- add guest to mobile app push notifications for local events

We could add logic to perform these actions to the `checkIn()` method, but that
would require that the `HotelRoom` class now be explicitly aware of an email
system and a mobile app push notification system, neither of which should be
responsibilities of the hotel room itself. Our hotel room should be responsible
for its own actions, and then notify other entities that some things have
changed and let those other entities decide for themselves what to do about
those changes.

In order to make this possible, we need to:

- Create an interface called `RoomCheckinObserver` that every class that's
  interested in the room check-in events will implement. This will allow us to
  manage any observers no matter their actual class, as long as they implement
  this interface.
- Add methods to the `HotelRoom` class to add/remove observers
- Add logic to the `checkIn()` method to notify all the registered observers
- Add logic to the `HotelManager` class, where the hotel rooms are created, to
  add the appropriate observers for each room
- Create a `HotelEmailService` class that implements the `RoomCheckinObserver`
  interface
- Create a `HotelPushNotificationService` class that also implements the
  `RoomCheckinObserver` interface

First, let's create the `RoomCheckinObserver` interface:

```java
interface RoomCheckinObserver {
    public void update(Object updateInfo);
}
```

We then add methods to add/remove observers from a hotel room to the `HotelRoom`
class, and add logic to the `checkIn()` method to notify all the observers that
a check-in just occurred.

```java
class HotelRoom implements HotelRoomInterface {
    private List<RoomCheckinObserver> checkinObservers = new ArrayList<RoomCheckinObserver>();

    public void book(String guestName) {
        Logger.getInstance().log("Booked a room for " + guestName);
    }

    public void clean() {
        Logger.getInstance().log("Cleaned room");
    }

    public void addCheckinObserver(RoomCheckinObserver checkinObserver) {
        checkinObservers.add(checkinObserver);
    }

    public void removeCheckinObserver(RoomCheckinObserver checkinObserver) {
        checkinObservers.remove(checkinObserver);
    }

    public void checkIn(String guestName) {
        Logger.getInstance().log(guestName + "checked in");
        checkinObservers.forEach((checkinObserver -> checkinObserver.update(guestName)));
    }
}
```

Now any class that wants to be an observer for the room checkin process needs to
implement the `RoomCheckinObserver` interface. For our email and push
notification functionality, we create a `HotelEmailService` and a
`HotelPushNotificationService` class to handle each respectively:

```java
class HotelEmailService implements RoomCheckinObserver {
    public void update(Object guestName) {
        Logger.getInstance().log("Sent email update to " + guestName);
    }
}

class HotelPushNotificationService implements RoomCheckinObserver {
    public void update(Object guestName) {
        Logger.getInstance().log("Registered " + guestName + " for push notification updates");
    }
}
```

Finally, we need to add the observers to each room when it gets created, so we
update our `HotelManager` class since that's where our hotel rooms are currently
being created:

```java
public class HotelManager {
    public static void main(String[] args) {
       public class HotelManager {
          public static void main(String[] args) {
             Logger.getInstance().log("Managing hotel...");

             // create hotel rooms
             List<HotelRoom> hotelRooms = new ArrayList<HotelRoom>();
             // create hotel floors
             // add hotel rooms to hotel floors
             // take actions on rooms and floors and examine your output to ensure you implemented the desired
             // behaviors
             // create hotel email and notification services
             HotelEmailService emailService = new HotelEmailService();
             HotelPushNotificationService notificationService = new HotelPushNotificationService();
             // initialize hotel email and notification services
             // ...
             hotelRooms.forEach((hotelRoom) -> {
                hotelRoom.addCheckinObserver(emailService);
                hotelRoom.addCheckinObserver(notificationService);
             });
          }
       }
    }
}
```
