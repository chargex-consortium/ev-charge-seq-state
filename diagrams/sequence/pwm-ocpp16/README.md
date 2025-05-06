# SAE J1772 PWM Controlled Charging with OCPP 1.6J Sequence Diagram

![SAE J1772 PWM + OCPP 1.6J Sequence Diagram](./pwm-ocpp16.svg)

## Key Actors:
- **EV driver:** The person attempting to charge the vehicle. The driver plugs in the EVSE to the EV, performs authentication (e.g., using an RFID card), and unplugs the EVSE from the EV.
- **SAE J1772 EV:** Electric vehicle using analog control pilot signaling to charge via AC voltage.  
- **Charging Station (CS):** EVSE implementing both the analog control pilot (PWM) and the OCPP 1.6J client.  
- **OCPP 1.6J CSMS:** Back‑end Charge Station Management System orchestrating sessions via OCPP 1.6J.

---

## Sequence Overview
## 1. Initialization, Authentication, and Session Setup

### Establishing the Session and Notifying the CSMS
1. The EV driver plugs in the cable, changing the pilot state from A1→B1. 
2. The EVSE sends a `StatusNotification.req` (status = `Preparing`) to the CSMS, which acknowledges this message.  
**Insight:** The EVSE informs the CSMS that a charging session is beginning.

### Authorization
1. The driver presents an RFID (e.g., `CHARGEX123`), which the EVSE forwards to the CSMS via `Authorize.req`.  
2. The CSMS authorizes the driver by sending a `Authorize.conf` (status = `Accepted`). The EVSE turns on the control pilot oscillator (State B1→B2) with the default duty cycle.
**Insight:** RFID (ISO14443) and the accepted response code ensure only authorized users initiate charging. Additional OCPP 1.6J methods credit/debit card, mobile app, and start button.

### Session Start
1. The EV detects the EVSE is ready to charge and closes control pilot switch S2 (State B2→C2) to initiate charging.
2. The EVSE detects control pilot State C2 and closes its AC contactors to apply power to the EV's onboard charger.  
3. The EVSE sends a `StartTransaction.req` to the CSMS to officially begin the transaction.  The CSMS accepts the transaction with a subsequent StartTransaction.conf reply.
4. Optionally, the CSMS provides the EVSE with a Charging Profile via the `SetChargingProfile.req/res` message pair.  This Charging Profile does not take into account the needs of the EV driver but rather only the constraints of the grid.  The EVSE would change the control pilot duty cycle to adhere to the new charging profile.
5. The EVSE notifies the CSMS that charging has started by sending a `StatusNotification.req` (status = `Charging`)
**Insight:** Even though the CSMS can push a grid-driven charging profile to the EVSE via SetChargingProfile, the profile itself doesn’t reflect the EV driver’s departure or energy needs—instead, it strictly enforces grid constraints by adjusting the control-pilot duty cycle, meaning the vehicle must passively follow the grid’s limits rather than negotiate its own requirements.

## 2. Loop Charging (Ongoing Charging Process)

### AC Charging
1. The EVSE changes the control pilot duty cycle to adhere to the charging profile.  The duty cycle is a max amperage limit, not a setpiont. The EV follows this limit, ensuring it does not exceed the amperage limit set by the duty cycle of the control pilot.
2. The EVSE sends periodic meter readings to the CSMS via `MeterValueRequest`/`MeterValueResponse`.  

### Change Charge Profile (Initiated by CSMS)
1. The CSMS sends `SetChargingProfile.req` to the EVSE (e.g., due to updated power constraints) with a new `csChargingProfile`. The EVSE accepts the new Charging Profile in `SetChargingProfileResponse`. 
2. The EVSE changes the control pilot duty cycle to adhere to the new charging profile.
**Insight:** Ability to change Charge Profile ensures that charging can adapt dynamically to changes in grid conditions.


## 3. End Charging and Transaction Termination
1. The EV initiates termination of the charge session by opening control pilot switch S2 ((State C2→B2).
2. The EVSE detects this control pilot state change and opens the AC contactors.
3. The EVSE updates the CSMS of the new status by sending a `StatusNotification.req` (status = `SuspendedEV`)
4. The EV driver unplugs the charging cable from the EV, triggering a control pilot transition (State B2→A2).
5. The EVSE shuts off the control pilot oscillator (State A2→A1).
6. The EVSE ends the OCPP transaction by sending a `StopTransaction.req`.   
7. The EVSE sends `StatusNotification.req` (status = `Available`).
**Insight:** The careful sequence of messages and actions ensures that all parties are informed of the session’s closure and that the charging station resets for the next user.

---

## References
- [SAE J1772](https://doi.org/10.4271/J1772_202401)  
- [OCPP 1.6J](https://openchargealliance.org/protocols/open-charge-point-protocol/#OCPP1.6)  
- PlantUML source: `pwm-ocpp16.puml`

