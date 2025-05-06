# ISO 15118-2 Controlled Charging with OCPP 1.6J Sequence Diagram

![ISO 15118-2 Controlled Charging with OCPP 1.6J](./iso15118_2-ocpp16.png)

## Key Actors:
- **EV driver:** The person attempting to charge the vehicle. The driver plugs in the EVSE to the EV, performs authentication (e.g., using an RFID card), and unplugs the EVSE from the EV.
- **ISO 15118-2 EV:** The EV in this example is an ISO 15118-2 EV capable of smart charge scheduling.
- **EVSE:** The EVSE interfaces with both the EV (using ISO 15118-2) and the CSMS (using OCPP 1.6J).
- **CSMS:** Charge Station Management System monitors, authorizes, and controls the charging process.
- **Secondary Actor (SA):** Supplies additional data (such as tariff tables or maximum power schedules). The Secondary Actor could be an Energy Management System (EMS) or grid operator.

---

## 1. Initialization, Authentication, and Session Setup

### Establishing the Session and Notifying the CSMS
1. The EV driver plugs in the cable, triggering the EV and EVSE to initiate communication.  
2. The EVSE sends a `StatusNotification.req` (status = `Preparing`) to the CSMS, which acknowledges this message.  
**Insight:** The EVSE informs the CSMS that a charging session is beginning.

### Protocol Negotiation and Service Discovery
1. The EV and EVSE exchange messages (`SupportedAppProtocolReq/Res`, `SessionSetupReq/Res`, `ServiceDiscoveryReq/Res`, `PaymentServiceSelectionReq/Res`) to agree on supported protocols and available services.  
**Insight:** These messages confirm that both sides support the necessary communication standards and charging setup, paving the way for secure and authenticated operations.

### Authorization
1. The EV sends an `AuthorizationReq` while the driver presents an RFID (e.g., `CHARGEX123`), which the EVSE forwards to the CSMS via `Authorize.req`.  
2. While authorization verification is occuring on the CSMS, the EVSE responds to each `AuthorizationReq` with a subsequent `AuthorizationRes` (`EVSEProcessing` = `Ongoing`).
3. After verification, an `Authorize.conf` (status = `Accepted`) authorizes the session. The EVSE then responds to the next `AuthorizationReq` with `EVSEProcessing` = `Finished`.
**Insight:** RFID (ISO14443) and the accepted response code ensure only authorized users initiate charging. Additional OCPP 1.6J methods credit/debit card, mobile app, and start button.

---

## 2. Controlled Charging

### EV Charging Parameter Discovery and Setting Charge Profile
1. The EV sends a `ChargeParameterDiscoveryReq` with parameters `EnergyTransferMode`, `EnergyTransferMode` and `EVChargeParameter` which contains the EV's max/min capabilities. Optionally, the EV may provide `DepartureTime` and `EAmount` (Energy needed by departure time).  Unfortunately, with OCPP 1.6J this information can not be sent to the CSMS.
2. The EVSE responds with a `ChargeParameterDiscoveryRes` with it's max/min limits (`EVSEChargeParameter`) and `EVSEProcessing` = `Ongoing` with no Seconday Actor Schedule List (`SAScheduleList`).  The EVSE will respond to all subsequent `ChargeParameterDiscoveryReq` messages with `EVSEProcessing` = `Ongoing` until the CSMS accepts the OCPP transaction and provides a Charging Profile.
3. The EVSE sends a `StartTransaction.req` to the CSMS to officially begin the transaction.  The CSMS accepts the transaction with a subsequent StartTransaction.conf reply.   
4.  The EVSE then sends an optional Tarrif Table utilizing the `DataTransfer.req/conf` message pair.   
5. The CSMS provides the EVSE with a Charging Profile via the `SetChargingProfile.req/res` message pair.  This Charging Profile does not take into account the needs of the EV driver but rather only the constraints of the grid. 
6. Upon reception of the `SetChargingProfile.req` from the CSMS, the EVSE replies to the next `ChargeParameterDiscoveryReq` with a `ChargeParameterDiscoveryRes` with `EVSEProcessing` = `Finished` and the subsequent Charging/Tarrif Profile (`SAScheduleList`).
**Insight:** This is deemed "Controlled" Charging because the needs of the driver is not taken into account by the CSMS/SA.

### Charge Profile Finalization
1. Upon reception of the `ChargeParameterDiscoveryRes` with the `SAScheduleList`, the EV determines its schedule and sends `PowerDeliveryReq` (ChargeProgress = `Start`).
2. The EV close switch S2 in the control pilot circuit (State C) and the EVSE closes its contactors.  
3. The EVSE notifies the CSMS that charging has started by sending a `StatusNotification.req` (status = `Charging`)
**Insight:** The EV's planned Charge Profile is never sent to the CSMS and SA, the EVSE is aware of the planned charge profile.

---

## 3. Loop Charging (Ongoing Charging Process)

### AC Charging
- EV periodically sends `ChargingStatusReq`; EVSE replies with `ChargingStatusRes` (includes `EVSEMaxCurrent`, status).  
- EVSE reports energy and meter values to CSMS via `MeterValueRequest`/`MeterValueResponse`.
**Insight:** The EV does not provide SOC or other EV related information in the `ChargingStatusReq`, the frequency (i.e. how often this message is sent to the EVSE) is also EV OEM dependent, which could limit how quickly an EV responds to a new max current limit or schedule renegotiation request. 

### DC Charging
- The EV sends `CurrentDemandReq` within every 250 ms to maintain stable and real-time communication status with the EVSE. The `CurrentDemandReq` message supplies the EVSE with information such as the EV status, SOC, target current and voltage, maximum limits and present current and voltage.    
- The EVSE responds with `CurrentDemandRes` (present voltage/current, limits, status).
**Insight:** The DC charging loop is crucial for precise control, ensuring that the EVSE is delivering the EV’s requested current.

---

## 4. Change Charge Profile

### Triggering a Charge Profile Change
- **By CSMS:** Sends `SetChargingProfile.req` to EVSE (e.g., due to updated tariffs or power constraints) with a new `csChargingProfile`. EVSE replies in `ChargingStatusRes` with `EVSENotification` = `ReNegotiation`, prompting EV to send updated `ChargeParameterDiscoveryReq` message.  
- **By EV:** Sends `PowerDeliveryReq` (`chargeProcess = Renegotiate`) if departure time changes or energy needs of the driver changes.

### Implementing the Charge Profile Change
1. EV re-initiates discovery (`ChargeParameterDiscoveryReq/Res`).  
2. EVSE sends updated `SAScheduleList` or same if not triggered by CSMS via `ChargeParameterDiscoveryRes`.  
3. EV adjusts schedule and sends new `PowerDeliveryReq` (ChargeProgress = `Start`); may pause/stop.
**Insight:** Ability to change Charge Profile ensures that charging can adapt dynamically to changes in grid conditions or pricing.

---

## 5. End Charging and Transaction Termination
1. The EV initiates termination by sending `PowerDeliveryReq` (chargeProcess = `Stop`), EVSE opens contactors.  
2. The EV sends `SessionStopReq` to end communication session; EVSE confirms with `SessionStopRes`.  
3. The EVSE updates the CSMS of the new status by sending a `StatusNotification.req` (status = `SuspendedEV`) 
4. The EVSE ends the OCPP transaction by sending a `StopTransaction.req`.   
4. After driver unplugs, EVSE sends `StatusNotification.req` (status = `Available`).
**Insight:** The careful sequence of messages ensures that all parties are informed of the session’s closure and that the charging station resets for the next user.

---

## References
- [ISO 15118-2](https://www.iso.org/standard/55366.html)  
- [OCPP 1.6J](https://openchargealliance.org/protocols/open-charge-point-protocol/#OCPP1.6)  
- PlantUML source: `iso15118_2-ocpp16.puml`

