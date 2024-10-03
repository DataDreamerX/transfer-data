TimeIndex = 
ADDCOLUMNS (
    CALENDAR(MIN('YourTable'[Timestamp]), MAX('YourTable'[Timestamp])),
    "Time", TIME(HOUR('YourTable'[Timestamp]), MINUTE('YourTable'[Timestamp]), 0)
)

FilledValue = 
IF (
    ISBLANK ( SUM('YourTable'[YourValueColumn]) ),
    0,
    SUM('YourTable'[YourValueColumn])
)

