# Events Definition

| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| tableOpened     | [Table Opened Event](#tableopened-event) | Information about the event when the table was opened #tableOpenedEvent (optional)|
| welletCodeAdded | [Wellet Code Added Event](#welletcodeadded-event)  | Information about the event when the Wellet code is added to the table (optional)|
| tableClosed | [Table Closed Event](#tableclosed-event)  | Information about the event when the table was closed (optional)|
| tablePaid | [Table Paid Event](#tablepaid-event)   | Information about the event when the table was paid (optional)|

## tableOpened event
| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| date     | date | Timestamp for this event. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time) |
| userId | string  | User Id that performed the event |
| userFullName | string  | Full Name of the user that performed the event |

## welletCodeAdded event
| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| date     | date | Timestamp for this event. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time) |
| userId | string  | User Id that performed the event |
| userFullName | string  | Full Name of the user that performed the event |

## tableClosed event
| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| date     | date | Timestamp for this event. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time) |
| userId | string  | User Id that performed the event |
| userFullName | string  | Full Name of the user that performed the event |

## tablePaid event
| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| date     | date | Timestamp for this event. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time) |
| userId | string  | User Id that performed the event |
| userFullName | string  | Full Name of the user that performed the event |
