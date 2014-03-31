# APIProxy to server protocol

 This file specifies the wire protocol used for vehicles/GCS connection to the dronehub server.  Supporting documents
 are available, but it is intended that _this file_ should be self describing enough for a developer to be able to
 write a client adapter that can connect to the server.
 
 A basic understanding of Protocol Buffers is needed to understand this protocol.  The actual .proto file is located in
 src/main/protobuf/webapi.proto.  For more information see:
 https://developers.google.com/protocol-buffers/docs/overview
 
 This protocol is currently a draft and encryption is not yet supported.  A future variant 
 of this protocol will use SASL encapsulation for encryption 
 (http://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).
 A UDP variant may also be released at some point.

## Connection establishment
 * Clients should open a TCP connection to port 5555 on <server name tbd> 
 * Once the connection is open, the client should send an Envelope containing a LoginMsg (see documentation below)
 * The server will respond with an Envelope containing a LoginResponseMsg
 * Once login completes successfully client can send other message types (see documention for Envelope)
 
## Wire encoding
 * Messages are always sent/received using the protobuf writeDelimited/readDelimited functions
 * Every message sent over the wire is an Envelope (see documentation below)
 * Envelope is a 'variant record' and should contain one (and only one) Message 

 
## Message flows
 
### Account creation message flow (optional)
 * Client sends LoginMsg(with code = CHECK_USERNAME) as user is picking a username.
 * Server responds with LoginResponseMsg to indicate if username is available
 * Client sends LoginMsg(with code = CREATE) and the selected username and password
 * Server responds with LoginResponseMsg (if server populates message, the client should show the message to the user)

### Normal message flow for 'live' flights
 * Client sends LoginMsg with a valid username and password
 * Server responds with LoginResponseMsg (if server populates message, the client should show the message to the user)
 * (User arms vehicle)
 * Server responds with MissionResponseMsg
 * Client sends SetVehicleMsgs (one per vehicle the client is talking with).  These messages will indicate if mavlink 
   from the server is accepted.
 * Server does not respond
 * Client sends StartMissionMsg (must be sent _AFTER_ SetVehicleMsgs)
 * Client sends numerous MavlinkMsgs - ideally for all mavlink, but if the TCP link begins to back up client can use 
   heuristics to drop packets
 * If server wants to send commands/queries to vehicles, it can send MavlinkMsgs to the client
 * (User chooses to end the flight/disarm vehicle etc...)
 * Client sends StopMissionMsg
 * Server responds with MissionResponseMsg
 * Client disconnects

### Normal message flow for batch upload of old flights
(used if internet connectivity was not available while GCS was connected to vehicle)
  * This message flow is the same as the 'live' flight except the SetVehicleMsgs will indicate 
    that vehicle can not be contacted by server
 
# For more information 
See developer documentation at https://docs.google.com/document/d/1ihKneLwA4hXmKS1W2pbG9lty_EAwbmy0giusUwQ8dto/edit
 
