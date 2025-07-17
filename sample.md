```plantuml
@startuml
!includeurl https://raw.githubusercontent.com/proveo-ca/identity/refs/heads/main/proveo.iuml

actor Guest as G 
actor Host as H

note right of G
  Design a system for sending real-time toll payment notifications 
  when their vehicles pass through toll booths.
  Assume vehicle detection and toll calculation are already handled by 
  an upstream system, which inputs events in a service.
  
  The system must:
  
  - Receive toll charge events (vehicle ID, timestamp, toll amount, location)
  - Notify the user associated with the vehicle via email and/or SMS
  - Ensure at-least-once delivery of the notification
  - Persist the toll charge record for future reference

end note

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
@enduml
```
