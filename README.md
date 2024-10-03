FrequencyTable = 
SUMMARIZE(
    'YourGroupedTable', 
    'YourGroupedTable'[RequestCount], 
    "SessionCount", COUNT('YourGroupedTable'[SessionID])
)
