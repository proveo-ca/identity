Design a system for sending real-time toll payment notifications to users when their vehicles pass through toll booths.
Assume vehicle detection and toll calculation are already handled by an upstream system, which sends events to your
service.

The system must:

- Receive toll charge events `(vehicle ID, timestamp, toll amount, location)`
- Notify the user associated with the vehicle via email and/or SMS
- Ensure at-least-once delivery of the notification
- Persist the toll charge record for future reference

```plantuml
<style>
  root {
    FontName Padauk;
    BackgroundColor #FAFAFA;
  }
  component {
    BackgroundColor #31176B;
    b1a8d4
    BorderColor #64B5F6;
    FontColor #FAFAFA;
    FontSize 14;
    RoundCorner 10;
    Padding 8;
    Shadowing 2;
  }

  node {
    BackgroundColor #FAFAFA;
    BorderColor #78909C;
    FontSize 14;
    RoundCorner 8;
    Padding 10;
    Shadowing 3;
  }

  database {
    BackgroundColor #888;
    BorderColor #66BB6A;
    FontColor #FAFAFA;
    FontSize 14;
    RoundCorner 8;
    Shadowing 2;
  }

  actor {
    BackgroundColor #AA0A7D;
    BorderColor #FF9800;
    FontColor #AA0A7D;
    FontSize 14;
    RoundCorner 8;
    Padding 6;
  }

  queue {
    BackgroundColor #EFA63D;
    FontColor #121212;
  }

  arrow {
    LineColor #444;
    LineThickness 2;
  }

  note {
    BackgroundColor #AA0A7D;
    BorderColor #FDD835;
    FontColor #FAFAFA;
  }

  cloud {
    LineColor #31176B;
  }

  frame {
    LineColor #31176B;
  }

  folder {
    LineColor #EFA63D;
  }
</style>

!define to [thickness=3]
!define err [#E20024]
!define db [#585858]
!define errLabel(x) <color:#E20024><b>x</b></color>
!define dbLabel(x) <color:#585858><b>x</b></color>

actor Guest as G 
actor Host as H

frame "Mobile Device" as UI {
    component App
    component Email
    component SMS
} 
node "Vehicle" as V
node "REST Service" as API {
  component "Validation Layer"
  database "Primary" as DB
}

cloud "Upstream System" as US {
  component "Vehicle Detection"
  component "Toll Calculation"
}

cloud Microservices {
  frame "Vehicle Services" as VS {
    component "Toll Charge Handler"
  }
  frame "Notification Services" as NS {
    component "SMS/Email Service"
    component "Push Notifications"
  }
}

cloud Monitoring as M {
  frame "Monitoring Services" {
    component Telemetry
    component Alerts
  }
  database Logs
}
queue "Vehicle Events" as VQ
queue "Notification Events" as NQ
frame "Dead Letter Queues" as DQ {
  queue "Vehicle Event DLQ" as VDQ
  queue "Notification Event DLQ" as NDQ
}

G -to> V  : " Triggers Events (in the Background)"
V -to-> US : " Sends requests Upstream"
US -to> VQ :  " TollEvent(vehicleId, time, amount, location)"
VQ -to-> API: " Consume Event"
API -to> API: " Validates Event: Guest Reservation/time, etc"
API -to--> VS : " Forward Event: Toll Charges"
VS -[#gray]left-> API : dbLabel(" Persisting Toll Charges")
VS -to-> NQ : " Enqueue Notification"
NQ -to> NS : " Broadcast Notification"
NS -to-> UI : " Receives Notification"
API -err-> DQ : errLabel(" Event Validation Failed")
NS -err-> DQ : errLabel("Failed Notifications")
API <--> DQ : " Retry strategies (e.g. fallback SMS, etc)"

API <.db.> UI : dbLabel(" Sync State")
API -err-> M : errLabel(" Error Reporting")
UI -err-> M : errLabel(" Error Reporting")
M <.db. DQ : dbLabel("Persists Errors")
UI -to-> H

note left of G: "Rents a Vehicle,\n drives in a highway"
note top of H: "New \nNotification"
```
