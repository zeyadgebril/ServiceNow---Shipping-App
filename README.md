<h1 align="center">Shipping APP Solution</h1>
<p align="center">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/logo.png" width="180"/>
</p>




![App Screenshot](https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-13%20044649.png)

## Summary

A dynamic ServiceNow-based shipping application that enables users to submit shipping requests, enter shipment details, and track their deliveries in real time through a live map tracker, while providing a streamlined and user-friendly experience.


## Key Roles

- Customer  
- Pricing Responsibility  
- Logistics  

Each role is responsible for a specific stage in the request lifecycle.


## Customer

Initiates service requests and monitors their status.


![App Screenshot](https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-01%20005035.png)

**Shipping Status**
---
<div align="center">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-15%20120819.png" width="30%" style="margin: 10px;" alt="Calculating Price" />
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-15%20120833.png" width="30%" style="margin: 10px;" alt="Waiting for Approval" />
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-15%20120844.png" width="30%" style="margin: 10px;" alt="Processing" />
</div>

<br/>

<div align="center">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-15%20120856.png" width="30%" style="margin: 10px;" alt="Booked" />
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-15%20120922.png" width="30%" style="margin: 10px;" alt="On the Way" />
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-15%20120938.png" width="30%" style="margin: 10px;" alt="Delivered" />
</div>


<h2 align="center">🔴 Live Map Tracking for Customers</h2>
<p align="center">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-10%20002714.png" alt="AIR Tracking" width="48%" style="margin-right:2%">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-28%20232121.png" alt="SEA Tracking" width="48%">
</p>


**Automated Status Email**
---
<p align="center">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-28%20220919.png" width="180"/>
</p>




## Pricing Responsibility
 Validates requests and assigns pricing before approval.

![App Screenshot](https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-14%20165429.png)

## Logistics
 Manages shipment execution and updates logistics details.

![App Screenshot](https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-14%20025345.png)

## API Reference 🦾

### 🌐 API Base URL (Flight Tracking 🛩)
`https://fr24api.flightradar24.com`

| Headers | Type   |
| :-------- | :------- |
| `Accept`      | `application/json` |
| `Accept-Version`      | `v1` |
| `Authorization`      | `Bearer <Token>` |


#### Get real-time flight positions with detailed information


```http
  GET /api/live/flight-positions/full
```


| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `flights`      | `string` | **Required**. Flight number for tracking Airplane|


### 🌐 API Base URL (Vessel Tracking 🚢)
`https://api.myshiptracking.com`


| Headers | Type   |
| :-------- | :------- |
| `Accept`      | `application/json` |
| `Authorization`      | `Bearer <Token>` | 

#### Get Vessel Search


```http
  GET /api/v2/vessel/search
```



| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `name`      | `string` | **Required**. vessel name for tracking|


#### Get Vessel Status


```http
  GET /api/v2/vessel
```



| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `mmsi`      | `string` | Vessel tracking ID|
| or  `imo`      | `string` | Vessel tracking ID|



# Shipping API Integration

This class is used in Script Includes to provide methods for fetching shipping information via **Air and Sea APIs**.


```javascript
var ShippingAPIRequest = Class.create();
ShippingAPIRequest.prototype = {
    initialize: function() {},
    airShippingAPI: function(flightnumber) {
        var request = new sn_ws.RESTMessageV2("AIR Current Shipping", "GET Flight Full Postion"); 
        request.setStringParameter("flights", flightnumber);
        var response = request.execute();
        var status = response.getStatusCode();
        if (status == 200) {
            return JSON.parse(response.getBody());
        } else {
            gs.error("API Call Failed. Status: " + status);
            return null;
        }
    },
    seaShippingSearchAPI: function(vessel_name) {
        var request = new sn_ws.RESTMessageV2("SEA Current Shipping", "GET Vessel Search");
        request.setStringParameter("name", vessel_name);
        var response = request.execute();
        var status = response.getStatusCode();
        if (status == 200) {
            return JSON.parse(response.getBody());
        } else {
            gs.error("API Call Failed. Status: " + status);
            return null;
        }
    },
    seaShippingStatusAPI: function(mmsi, imo) {
        if (mmsi) {
            var reqMmsi = new sn_ws.RESTMessageV2("SEA Current Shipping", "GET Vessel Status - mmsi");
            reqMmsi.setStringParameter("mmsi", mmsi);
            var resMmsi = reqMmsi.execute();
            if (resMmsi.getStatusCode() === 200) {
                var bodyMmsi = JSON.parse(resMmsi.getBody());
                if (bodyMmsi && bodyMmsi.data) {
                    return bodyMmsi;
                }
            }
        }

        if (imo) {
            var reqImo = new sn_ws.RESTMessageV2("SEA Current Shipping", "GET Vessel Status - imo");
            reqImo.setStringParameter("imo", imo);
            var resImo = reqImo.execute();
            if (resImo.getStatusCode() === 200) {
                var bodyImo = JSON.parse(resImo.getBody());
                if (bodyImo && bodyImo.data) {
                    return bodyImo;
                }
            }
        }

        gs.info("No vessel found for MMSI: " + mmsi + " or IMO: " + imo);
        return null;
    },
    type: 'ShippingAPIRequest'
};
```

## Scheduled Jobs for Air & Sea Shipping Tracking

These scripts are used in **Scheduled Jobs** to automatically track Air and Sea shipments using the `ShippingAPIRequest` class.

**Sea Shipping Scheduled Job**
---
```javascript
(function executeSeaShippingScheduledJob() {
    gs.info("Sea Shipping Job has started");
    var plus30Min = new GlideDateTime();
    plus30Min.addSeconds(1800); // 30 minutes ahead

    var shippingGR = new GlideRecord("x_1850691_zeyad_0_approved_request");
    shippingGR.addQuery("shipping_state", "booked").addOrCondition('shipping_state', 'on_the_way');
    shippingGR.addQuery("shipping_type", "sea");
    shippingGR.addQuery("shipping_start_time", "<=", plus30Min);
    shippingGR.query();

    while (shippingGR.next()) {
        gs.info("script started");
        try {
            var shippingAPI = new ShippingAPIRequest();
            if (gs.nil(shippingGR.imo) && gs.nil(shippingGR.mmsi)) {
                var vesselSearch = shippingAPI.seaShippingSearchAPI(shippingGR.vessel_name.trim().toUpperCase());
                if (vesselSearch && vesselSearch.data.length > 0) {
                    var vessel = vesselSearch.data[0];
                    shippingGR.setValue("mmsi", vessel.mmsi);
                    shippingGR.setValue("imo", vessel.imo);
                    shippingGR.setValue("vessel_type", vessel.vessel_type);
                    shippingGR.setValue("vessel_flag", vessel.flag);
                    shippingGR.setValue("starting_area", vessel.area);
                    shippingGR.update();
                } else {
                    gs.warn("No vessel found for: " + shippingGR.vessel_name);
                    continue;
                }
            } else {
                var result = shippingAPI.seaShippingStatusAPI(shippingGR.mmsi, shippingGR.imo);
                var responce = result.data;

                if (!responce || !responce.lat || !responce.lng) {
                    gs.warn("Incomplete API response for: " + shippingGR.number);
                    continue;
                }

                let latitude = responce.lat;
                let longitude = responce.lng;

                if (shippingGR.shipping_state !== 'on_the_way') {
                    shippingGR.setValue('shipping_state', 'on_the_way');
                }

                shippingGR.setValue("shipping_location", latitude + " , " + longitude);

                var oldGeoGR = new GlideRecord("geo_history");
                oldGeoGR.addQuery("x_1850691_zeyad_0_request_id", shippingGR.sys_id);
                oldGeoGR.addQuery("latest_record", true);
                oldGeoGR.query();
                while (oldGeoGR.next()) {
                    oldGeoGR.setValue("latest_record", false);
                    oldGeoGR.update();
                }

                var geoLocationGR = new GlideRecord('geo_history');
                geoLocationGR.initialize();
                geoLocationGR.setValue("x_1850691_zeyad_0_request_id", shippingGR.sys_id);
                geoLocationGR.setValue("latitude", latitude);
                geoLocationGR.setValue("longitude", longitude);
                geoLocationGR.setValue("location_timestamp", new GlideDateTime(responce.received));
                geoLocationGR.setValue("latest_record", true);
                geoLocationGR.insert();

                var stopedVeselCounter = shippingGR.empty_api_count || 0;
                if (responce.nav_status == 5 && stopedVeselCounter >= 12) {
                    gs.info("vessel have arrived: " + shippingGR.number);
                    shippingGR.setValue("shipping_state", 'delivered');
                    shippingGR.setValue("shipping_end_time", new GlideDateTime(responce.received));
                    stopedVeselCounter++;
                    shippingGR.empty_api_count = stopedVeselCounter;
                } else {
                    shippingGR.empty_api_count = 0;
                    shippingGR.update();
                }

                shippingGR.update();
            }

        } catch (ex) {
            gs.error('Scheduled Job Error for shipping ' + shippingGR.number + ' : ' + ex);
        }
    }
})();
```
**Air Shipping Scheduled Job**
---
```javascript
(function executeAirShippingScheduledJob() {
    gs.info("Air Shipping Job has started");

    var shippingGR = new GlideRecord("x_1850691_zeyad_0_approved_request");
    shippingGR.addQuery("shipping_state", "booked").addOrCondition('shipping_state', 'on_the_way');
    shippingGR.addQuery("shipping_type", "air");
    shippingGR.query();

    while (shippingGR.next()) {
        gs.info("Processing shipping: " + shippingGR.number);
        try {
            var offsetHours = parseInt(shippingGR.shipping_origin_timezone, 10) || 0;
            var shippingStart = shippingGR.shipping_start_time;
            var shippingStartAdjusted = new GlideDateTime(shippingStart);
            shippingStartAdjusted.addSeconds(-offsetHours * 3600);
            shippingStartAdjusted.addSeconds(-900); // 15 minutes before
            var nowGMT = new GlideDateTime();

            if (shippingStartAdjusted.compareTo(nowGMT) > 0) {
                gs.info("Skipping future shipment: " + shippingGR.number);
                continue;
            }

            var shippingAPI = new ShippingAPIRequest();
            var result = shippingAPI.airShippingAPI(shippingGR.flight_number);
            var flight = result.data[0];

            if (!result) {
                gs.warn("No API response for - " + shippingGR.number);
                continue;
            }

            if (result.data.length > 0) {
                let latitude = flight.lat;
                let longitude = flight.lon;

                if (shippingGR.shipping_state !== 'on_the_way') {
                    shippingGR.setValue('shipping_state', 'on_the_way');
                }

                shippingGR.setValue("shipping_location", latitude + " , " + longitude);
                shippingGR.update();

                var oldGeoGR = new GlideRecord("geo_history");
                oldGeoGR.addQuery("x_1850691_zeyad_0_request_id", shippingGR.sys_id);
                oldGeoGR.addQuery("latest_record", true);
                oldGeoGR.query();
                while (oldGeoGR.next()) {
                    oldGeoGR.setValue("latest_record", false);
                    oldGeoGR.update();
                }

                var geoLocationGR = new GlideRecord('geo_history');
                geoLocationGR.initialize();
                geoLocationGR.setValue("x_1850691_zeyad_0_request_id", shippingGR.sys_id);
                geoLocationGR.setValue("latitude", latitude);
                geoLocationGR.setValue("longitude", longitude);
                geoLocationGR.setValue("location_timestamp", new GlideDateTime());
                geoLocationGR.setValue("latest_record", true);
                geoLocationGR.insert();

                shippingGR.empty_api_count = 0;
                shippingGR.update();
            } else {
                var emptyCount = shippingGR.empty_api_count || 0;
                emptyCount++;
                shippingGR.empty_api_count = emptyCount;
                shippingGR.update();

                if (emptyCount >= 3) {
                    gs.info("Shipment delivered: " + shippingGR.number);
                    shippingGR.setValue("shipping_state", "delivered");
                    shippingGR.update();
                }
            }

        } catch (ex) {
            gs.error('Scheduled Job Error for shipping ' + shippingGR.number + ' : ' + ex);
        }
    }
})();
```


**Result**
-----
<p align="center">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-28%20231058.png" alt="Image 1" width="32%" style="margin-right:1%">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-28%20230610.png" alt="Image 2" width="32%" style="margin-right:1%">
  <img src="https://raw.githubusercontent.com/zeyadgebril/ServiceNow---Shipping-App/refs/heads/master/img/Screenshot%202026-01-28%20230634.png" alt="Image 3" width="32%">
</p>
