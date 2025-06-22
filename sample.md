Design a system for sending real-time toll payment notifications to users when their vehicles pass through toll booths. Assume vehicle detection and toll calculation are already handled by an upstream system, which sends events to your service.

The system must:
- Receive toll charge events `(vehicle ID, timestamp, toll amount, location)`
- Notify the user associated with the vehicle via email and/or SMS
- Ensure at-least-once delivery of the notification
- Persist the toll charge record for future referenceDesign a system for sending real-time toll payment notifications to users when their vehicles pass through toll booths. Assume vehicle detection and toll
  calculation are already handled by an upstream system, which sends events to your service.

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
    BackgroundColor #CB159A;
    BorderColor #FF9800;
    FontColor #CB159A;
    FontSize 14;
    RoundCorner 8;
    Padding 6;
  }

  queue {
    BackgroundColor #F2C633;
    FontColor #121212;
  }

  arrow {
    LineColor #444;
    LineThickness 2;
  }

  note {
    BackgroundColor #CB159A;
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
    LineColor #F2C633;
  }
</style>

!define to [thickness=3]
!define err [#red]
!define db [#gray]
!define errLabel(x) <color:red><b>x</b></color>
!define dbLabel(x) <color:gray><b>x</b></color>

actor Guest as G 
actor Host as H


node "Mobile Device" as UI {
    component App
    component Email
    component SMS
} 
node "Vehicle" as V
node "REST Service" as API {
    database "Primary" as DB
}

cloud Microservices {
    node "Vehicle Service" as VS {
        component "Toll Charge Handler"
    }
    node "Notification Service" as NS
}

cloud Monitoring as M {
    node "Monitoring Service" {
        component Telemetry
        component Alerts
        database "Logs Persistence"
    }
}
queue "Vehicle Events" as VQ
queue "Notification Events" as NQ
frame "Dead Letter Queues" as DQ {
  queue "Vehicle Event DLQ" as VDQ
  queue "Notification Event DLQ" as NDQ
}

G -to> V  : " Triggers Events (in the Background)"
V -to> VQ : "  TollEvent(vehicleId, time, amount, location)"
VQ -to-> API: " Consume Event"
API -to-> API: " Validates Event: Guest Reservation/time, etc"
API -to--> VS : " Forward Event: Toll Charges"
VS -to-> NQ : " Enqueue Notification"
NQ -to-> NS : " Broadcast Notification"
NS -to-> UI : " Receives Notification"
API -err-> DQ : errLabel(" Event Validation Failed")
NS -err-> DQ : errLabel("Failed Notifications")
VS -[#gray]left-> API : dbLabel(" Persisting Toll Charges")
API <--> DQ : " Retry strategies (e.g. fallback SMS, etc)"

API <.db.> UI : dbLabel(" Sync State")
API -err-> M : errLabel(" Error Reporting")
UI -err-> M : errLabel(" Error Reporting")
M <.db. DQ : dbLabel("Persists Errors")
UI -to-> H : " Reads Notification"

note bottom of G: "Rents a Vehicle,\n drives in a highway"
note bottom of H: "Reads \nNotification"
```
