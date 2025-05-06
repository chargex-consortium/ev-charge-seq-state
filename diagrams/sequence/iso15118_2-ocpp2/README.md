# ISO 15118-2 HLC Optimized Charge Scheduling with OCPP 2.0.1 Sequence Diagram

![ISO 15118-2 HLC Optimized Charge Scheduling with OCPP 2.0.1](./iso15118_2-ocpp2.png)

## Key Actors:
- **EV driver:** The person attempting to charge the vehicle. The driver plugs in the EVSE to the EV, performs authentication (e.g., using an RFID card), and unplugs the EVSE from the EV.
- **ISO 15118-2 EV:** The EV in this example is an ISO 15118-2 EV capable of smart charge scheduling.
- **EVSE:** The EVSE interfaces with both the EV (using ISO 15118-2) and the CSMS (using OCPP 2.0.1).
- **CSMS:** Charge Station Management System monitors, authorizes, and optimizes the charging process.
- **Secondary Actor (SA):** Supplies additional data (such as tariff tables or maximum power schedules). The Secondary Actor could be an Energy Management System (EMS) or grid operator.

---

## 1. Initialization, Authentication, and Session Setup

### Establishing the Session and Notifying the CSMS
1. The EV driver plugs in the cable, triggering the EV and EVSE to initiate communication.  
2. The EVSE sends a `StatusNotificationRequest` (state: “Occupied”) and a `TransactionEventRequest` (eventType = `Started`, chargingState = `EVConnected`, triggerReason = `CablePluggedIn`) to the CSMS, which acknowledges these messages.  
**Insight:** The trigger reason “CablePluggedIn” and state “Occupied” ensure the CSMS knows a charging session is beginning.

### Protocol Negotiation and Service Discovery
1. The EV and EVSE exchange messages (`SupportedAppProtocolReq/Res`, `SessionSetupReq/Res`, `ServiceDiscoveryReq/Res`, `PaymentServiceSelectionReq/Res`) to agree on supported protocols and available services.  
**Insight:** These messages confirm that both sides support the necessary communication standards and charging setup, paving the way for secure and authenticated operations.

### Authorization
1. The EV sends an `AuthorizationReq` while the driver presents an RFID (e.g., `CHARGEX123`), which the EVSE forwards to the CSMS via `AuthorizeRequest`.  
2. While authorization verification is occuring on the CSMS, the EVSE responds to each `AuthorizationReq` with a subsequent `AuthorizationRes` (`EVSEProcessing` = `Ongoing`).
2. After verification, an `AuthorizeResponse` and subsequent `AuthorizationRes` (ResponseCode = `Accepted`) authorizes the session. 
**Insight:** The use of the RFID (type ISO14443) and the accepted response code ensure that only authorized users can initiate charging.  Other means of authorization supported by OCPP 2.0.1 include: contract certificates (PnC), credit/debit card, mobile app (CSMS initiated), start button, and PIN-code.

---

## 2. Optimized Charge Scheduling

### EV Charging Parameter Discovery
1. The EV sends a `ChargeParameterDiscoveryReq` containing key parameters such as `EnergyTransferMode`, `DepartureTime`, `EAmount` (Energy needed by departure time) and `EVChargeParam` which contains the EV's max/min capabilities. 
2. The EVSE forwards these charging needs to the CSMS via a NotifyEVChargingNeedsRequest, and the CSMS acknowledges with a corresponding response.    
**Insight:** Parameters such as EAmount and DepartureTime are critical for the CSMS to generate an optimized charging schedule.

### Charge Profile Optimization and Finalization
1. The CSMS runs an internal optimization algorithm (possibly incorporating tariff tables and PmaxSchedules provided by the SA) to determine the available power schedule for the EV.  
2. If tariff tables are also utilized a separate DataTransferRequest is sent to the EVSE.   
3. The CSMS sends `SetChargingProfileRequest` with the available power profile. EVSE includes this in `ChargeParameterDiscoverRes` to EV.  
4. The EV uses this to determine its optimal charging schedule and sends `PowerDeliveryReq` (detailed profile, ChargeProgress = `Start`).  
5. The EVSE notifies the CSMS of the EV's charging schedule via `NotifyEVChargingScheduleRequest`; schedules may also go to SA.
**Insight:** The charging schedule (often a list of profile entries) informs the CSMS and SA the maximum power the EV will consume at each time slot to meet the EV’s needs optimally.

---

## 3. Loop Charging (Ongoing Charging Process)

### AC Charging
- The EV periodically sends `ChargingStatusReq`; EVSE replies with `ChargingStatusRes` (includes `EVSEMaxCurrent`, status).    
- The EVSE reports energy and meter values to CSMS via `MeterValueRequest`/`MeterValueResponse`.
**Insight:** The EV does not provide SOC or other EV related information in the `ChargingStatusReq`, the frequency (i.e. how often this message is sent to the EVSE) is also EV OEM dependent, which could limit how quickly an EV responds to a new max current limit or schedule renegotiation request. 

### DC Charging
- The EV sends `CurrentDemandReq` within every 250 ms to maintain stable and real-time communication status with the EVSE. The `CurrentDemandReq` message supplies the EVSE with information such as the EV status, SOC, target current and voltage, maximum limits and present current and voltage.    
- The EVSE responds with `CurrentDemandRes` (present voltage/current, limits, status).
**Insight:** The DC charging loop is crucial for precise control, ensuring that the EVSE is delivering the EV’s requested current.

---

## 4. Renegotiation of Charge Schedule

### Triggering a Renegotiation
- **By CSMS:** Sends `SetChargingProfileRequest` to EVSE (e.g., due to updated tariffs or power constraints) with a new `ChargingProfile`. EVSE replies in `ChargingStatusRes` with `EVSENotification` = `ReNegotiation`, prompting EV to send updated `ChargeParameterDiscoveryReq` with updated `DepartureTime` and `EAmount` values.  
- **By EV:** Sends `PowerDeliveryReq` (chargeProcess = `Renegotiate`) with updated `DepartureTime` and `EAmount` if departure time changes or energy needs of the driver changes.

### Implementing the Renegotiation
1. The EV re-initiates discovery (`ChargeParameterDiscoveryReq/Res`).  
2. The EVSE sends updated `SetChargingProfileRequest` to EV via `ChargeParameterDiscoverRes`.  
3. The EV adjusts schedule and sends new `PowerDeliveryReq` (ChargeProgress = `Start`); may pause/stop if needed.  
4. The EVSE notifies the CSMS via `NotifyEVChargingScheduleRequest`; schedules maybe sent to SA.
**Insight:** The renegotiation process ensures that charging can adapt dynamically to changes in grid conditions, pricing, or EV requirements.

---

## 5. End Charging and Transaction Termination
1. The EV initiates termination by sending `PowerDeliveryReq` (chargeProcess = `Stop`), EVSE opens contactors.  
2. The EV sends `SessionStopReq` to end communication session; EVSE confirms with `SessionStopRes`.  
3. The EVSE updates the CSMS: first `TransactionEventRequest` (EV still connected), then final `TransactionEventRequest` (eventType = `Ended`, chargingState = `Idle`).  
4. After driver unplugs, EVSE sends `StatusNotificationRequest` (state: `Available`).
**Insight:** The careful sequence of messages ensures that all parties are informed of the session’s closure and that the charging station resets for the next user.

## References
- [ISO 15118-2](https://www.iso.org/standard/55366.html)
- [OCPP 2.0.1 (IEC 63584)](https://openchargealliance.org/protocols/open-charge-point-protocol/#OCPP2.0.1)
- PlantUML source: `hlc-schedule.puml`

