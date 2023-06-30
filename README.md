http:
  - match:
      - uri:
          prefix: /websocket-path
    route:
      - destination:
          host: <destination-service>
    websockets: true
