import UIKit
import SocketIO
import ObjectMapper
import SwiftyJSON

//MARK:- Socket Connection Manager Protocol
protocol SocketConnectionDelegate: class {
    func onSocketConnected()
    func onSocketDisconnected()
    func oldMessages(messages:[LastMessage])
    func chatList(chatList:[ChatList])
    func recivedMessage(message:LastMessage?)
    func fetchUsersChatList()
    func roomOldMessages(messages:[LastMessage])
    func roomRecivedMessage(message:LastMessage?)
}
/**
 iNet : SocketConnectionManager
 
 This Class manages the socket operations for ChatViewController
 
 - Author:
 Danish Khan
 
 - Date:
 09/09/2021
 
 - Version:
 1.0
 */
class SocketConnectionManager{
    
    static let shared:SocketConnectionManager = SocketConnectionManager()
    var manager: SocketManager!
    var socket: SocketIOClient!
    private var appDelegate:UIApplicationDelegate! = nil
    weak var socketDelegate : SocketConnectionDelegate?
    
    private init() {
        self.appDelegate = UIApplication.shared.delegate
    }
    
    var localTimeZoneIdentifier: String { return TimeZone.current.identifier }
    
    func connectSocket(){
        manager = SocketManager(socketURL:  URL(string: Constants.SocketDetail.url)!, config: [.log(true),.compress, .extraHeaders(["Time-Zone": localTimeZoneIdentifier])])
        
        socket = manager.defaultSocket
        
        socket.connect()
        
        socket.on(clientEvent: .connect) {data, ack in
            print("socket Connected")
            if let user = CommonUtilities.getUserFromUserDefaults(){
                SocketConnectionManager.shared.socket.emit(Events.socketJoin.rawValue, user.id ?? -1)
            }
            
        }
        
        socket.on(clientEvent: .disconnect) { (data, ack) in
            self.socketDelegate?.onSocketDisconnected()
            Logger.log("socket disconnect")
        }
        
        socket.on(Listner.joinChat.rawValue) {data, ack in
            
            self.socketDelegate?.onSocketConnected()
            Logger.log("Join connected")
        }
        
        socket.on(Listner.oldMessages.rawValue) { data, ack in
            print("ON Recive messages")
            if let dataStringJson = data[0] as? [String:Any] {
                if let dict = dataStringJson["messages"] as? [[String:Any]] {
                    let messageObject = Mapper<LastMessage>().mapArray(JSONObject: dict)
                    self.socketDelegate?.oldMessages(messages: messageObject ?? [])
                }
            }
        }
        
        socket.on(Listner.roomOldMessage.rawValue) { data, ack in
            print("ON Recive messages")
            if let dataStringJson = data[0] as? [String:Any] {
                if let dict = dataStringJson["messages"] as? [[String:Any]] {
                    let messageObject = Mapper<LastMessage>().mapArray(JSONObject: dict)
                    self.socketDelegate?.roomOldMessages(messages: messageObject ?? [])
                }
            }
        }
        
        socket.on(Listner.socketJoin.rawValue) { data, ack in
            self.socketDelegate?.fetchUsersChatList()
            //NotificationCenter.default.post(name: .refreshChatUserList, object: nil)
        }
        
        socket.on(Listner.fireUserListEvent.rawValue) { data, ack in
            //self.socketDelegate?.fetchUsersChatList()
            NotificationCenter.default.post(name: .refreshChatUserList, object: nil)
        }
        
        socket.on(Listner.userChatList.rawValue) { data, ack in
            print("ON Recive Chat List")
            if let dataStringJson = data[0] as? [String:Any] {
                print("CHAT LIST: \(dataStringJson)")
                if let dict = dataStringJson["userList"] as? [[String:Any]] {
                    let messageObject = Mapper<ChatList>().mapArray(JSONObject: dict)
                    NotificationCenter.default.post(name: .chatUserList, object: messageObject)
                    self.socketDelegate?.chatList(chatList: messageObject ?? [])
                }
                if let archiveCount = dataStringJson["archive_count"] as? Int {
                    print("archive_count \(archiveCount)")
                    let archiveDict:[String: Int] = ["archive_count": archiveCount]
                    NotificationCenter.default.post(name: .archivedCount, object: nil,userInfo: archiveDict)
                }
            }
        }
        
        socket.on(Listner.archiveChatUserList.rawValue) { data, ack in
            print("ON Recive Chat List")
            if let dataStringJson = data[0] as? [String:Any] {
                if let dict = dataStringJson["userList"] as? [[String:Any]] {
                    let messageObject = Mapper<ChatList>().mapArray(JSONObject: dict)
                    NotificationCenter.default.post(name: .chatUserList, object: messageObject)
                    self.socketDelegate?.chatList(chatList: messageObject ?? [])
                }
                if let archiveCount = dataStringJson["archive_count"] as? Int {
                    print("archive_count \(archiveCount)")
                    let archiveDict:[String: Int] = ["archive_count": archiveCount]
                    NotificationCenter.default.post(name: .archivedCount, object: nil,userInfo: archiveDict)
                }
            }
        }
        
        socket.on(Listner.errorMessage.rawValue) { data, ack in
            if let dataStringJson = data[0] as? [String:Any] {
                print(dataStringJson)
                //if let dict = dataStringJson["message"] as? [String:Any] {
                // let messageObject = Mapper<ChatMessage>().map(JSON: dict)
                //self.socketDelegate?.onReceivedMessage(messageObj: messageObject!)
                //}
            }
        }
        
        socket.on(Listner.reciveMessage.rawValue) {data, ack in
            print("ON Recive Message")
            if let dataStringJson = data[0] as? [String:Any] {
                if let dict = dataStringJson["message"] as? [String:Any] {
                    let messageObject = Mapper<LastMessage>().map(JSON: dict)
                    self.socketDelegate?.recivedMessage(message: messageObject ?? nil)
                }
            }
        }
        
        socket.on(Listner.roomReciveMessage.rawValue) {data, ack in
            print("ON Recive Message")
            if let dataStringJson = data[0] as? [String:Any] {
                if let dict = dataStringJson["message"] as? [String:Any] {
                    let messageObject = Mapper<LastMessage>().map(JSON: dict)
                    self.socketDelegate?.roomRecivedMessage(message:messageObject ?? nil)
                }
            }
        }
    }
    
    func emit(eventName:String,params: [String : Any]) {
        SocketConnectionManager.shared.socket.emit(eventName, params)
    }
    
    func disconnectSocket(){
        self.socketDelegate?.onSocketDisconnected()
        socket.disconnect()
    }
    
    func reConnectSocket() {
        if !socket.status.active{
            connectSocket()
        }
    }
    
    func isSocketConnected() -> Bool {
        return socket.status.active
    }
}

enum Events: String{
    case socketJoin              = "joinSocket"
    case oldChatList             = "getChatUserList"
    case oldMessages             = "messageLoad"
    case sendMessage             = "sendMessage"
    case leaveChat               = "leave"
    case joinChat                = "join"
    case roomJoin                = "roomJoin"
    case roomLeave               = "roomLeave"
    case roomOldMessage          = "roomMessageLoad"
    case roomSendMessage         = "roomSendMessage"
    case getArchiveChatUserList  = "getArchiveChatUserList"
}

enum Listner: String{
    case userChatList            = "chatUserList"
    case oldMessages             = "messageList"
    case reciveMessage           = "reciveMessage"
    case joinChat                = "joinMessage"
    case errorMessage            = "errorMessage"
    case roomOldMessage          = "roomMessageList"
    case roomReciveMessage       = "roomReciveMessage"
    case archiveChatUserList     = "archiveChatUserList"
    case fireUserListEvent       = "fireUserListEvent"
    case socketJoin              = "joinSocket"
}
