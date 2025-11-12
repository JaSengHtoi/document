# 🚀 Speed Transfer Chat - How It Works

## 📋 Table of Contents
1. [Authentication Flow](#authentication-flow)
2. [Real-time Messaging Flow](#real-time-messaging-flow)
3. [File Upload Flow](#file-upload-flow)
4. [Group Creation Flow](#group-creation-flow)
5. [Admin Management Flow](#admin-management-flow)
6. [System Architecture](#system-architecture)

---

##  1. Authentication Flow

### **User Registration:**
```
User fills signup form → Flask /signup endpoint → Validate data
                                ↓
                        Generate unique User ID (USR000XXX)
                                ↓
                        Hash password (werkzeug)
                                ↓
                        Upload profile pic → Save to /static/uploads/profiles/
                                ↓
                        Store in SQLite (User table)
                                ↓
                        Create session
                                ↓
                        Redirect to /chat
```

### **User Login:**
```
User enters credentials → Flask /login endpoint → Check username/user_id
                                ↓
                        Validate password hash
                                ↓
                        Check ban status (active/blocked/temp)
                                ↓
                        Create session (username, user_id, role)
                                ↓
                        If admin → /admin | If user → /chat
```

### **Password Recovery:**
```
User enters User ID → Flask /forgot_password → Find user by user_id
                                ↓
                        Hash new password
                                ↓
                        Update database
                                ↓
                        Show success + username
                                ↓
                        Redirect to login
```

---

## 💬 2. Real-time Messaging Flow

### **Sending Messages:**
```
User types message → Click send → JavaScript sendMessage()
                                ↓
                        File attached? → Upload file first
                                ↓                ↓
                               Yes              No
                                ↓                ↓
                        /upload endpoint    Skip upload
                                ↓                ↓
                        Save to disk         ────┘
                                ↓
                        Get file_url
                                ↓
                    Socket.emit('send_message')
                                ↓
                        Flask SocketIO Handler
                                ↓
                        Generate message_id (UUID)
                                ↓
                        Save to SQLite (Message table)
                                ↓
                        Determine recipients
                                ↓
                ┌───────────────┴───────────────┐
                ↓                               ↓
            1-on-1 Chat                    Group Chat
                ↓                               ↓
        Emit to sender + recipient      Emit to all group members
                ↓                               ↓
        Socket.emit('receive_message')  Socket.emit('receive_message')
                                ↓
                        Client receives message
                                ↓
                        displayMessage() function
                                ↓
                        Append to chat UI
                                ↓
                        Scroll to bottom
```

### **Loading Chat History:**
```
User clicks contact → selectChat() → Socket.emit('get_messages')
                                ↓
                        Flask handler receives
                                ↓
                    Check if group or 1-on-1
                                ↓
                ┌───────────────┴───────────────┐
                ↓                               ↓
            Group Chat                      1-on-1 Chat
                ↓                               ↓
    Query: WHERE group_id = X      Query: WHERE (sender+recipient)
                ↓                               ↓
                └───────────────┬───────────────┘
                                ↓
                        Get all messages
                                ↓
                        Sort by created_at
                                ↓
                        Socket.emit('load_messages')
                                ↓
                        Client displays messages
```

### **Typing Indicator:**
```
User types → onkeypress event → Socket.emit('typing')
                                ↓
                        Flask handler
                                ↓
                        Identify recipient
                                ↓
                ┌───────────────┴───────────────┐
                ↓                               ↓
            Group Chat                      1-on-1 Chat
                ↓                               ↓
    Emit to all members              Emit to recipient only
                ↓                               ↓
                └───────────────┬───────────────┘
                                ↓
                        Show "Typing..." indicator
                                ↓
                        Auto-hide after 2 seconds
```

---

## 3. File Upload Flow

```
User selects file → handleFileSelect() → Validate file
                                ↓
                        Check file size (< 50MB)
                                ↓
                        Check file type (allowed extensions)
                                ↓
                        Show preview in UI
                                ↓
User clicks send → Create FormData → POST to /upload
                                ↓
                        Flask /upload handler
                                ↓
                        secure_filename() sanitization
                                ↓
                        Add timestamp to filename
                                ↓
                        Determine file category:
                                ↓
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
            Images          Videos          Files
    (png,jpg,jpeg,gif)  (mp4,avi,mov)   (pdf,doc,txt,zip)
                ↓               ↓               ↓
        /static/uploads/   /static/uploads/   /static/uploads/
           images/            videos/            files/
                ↓               ↓               ↓
                └───────────────┼───────────────┘
                                ↓
                        Save file to disk
                                ↓
                        Return file_url + file_type
                                ↓
                        Include in message data
                                ↓
                        Send via WebSocket
                                ↓
                        Store in database (Message table)
                                ↓
                        Display in chat:
                                ↓
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
            <img>           <video>         File card
             tag             tag           with icon
```

---

##  4. Group Creation Flow

```
User clicks "Create New Group" → openGroupModal()
                                ↓
                        Show modal with form
                                ↓
User enters group name + selects members
                                ↓
                        Submit form
                                ↓
                        Validate (name + at least 1 member)
                                ↓
                        POST to /create_group
                                ↓
                        Flask handler
                                ↓
                        Generate group_id (UUID)
                                ↓
                        Create Group record
                                ↓
                        Add creator as GroupMember
                                ↓
                        Loop through selected members
                                ↓
                        Create GroupMember records
                                ↓
                        Commit to database
                                ↓
                        Socket.emit('group_created')
                                ↓
                        Broadcast to all clients
                                ↓
                        Reload page to show new group
                                ↓
                        Group appears in Groups tab
```

---

## 🛡️ 5. Admin Management Flow

### **User Ban (Permanent):**
```
Admin clicks "Block" → POST /admin/block_user
                                ↓
                        Update User: status = 'blocked'
                                ↓
                        Set ban_type = 'permanent'
                                ↓
                        Commit to database
                                ↓
                        Socket.emit('user_blocked')
                                ↓
                        Broadcast notification
                                ↓
                        User can't login
```

### **User Ban (Temporary):**
```
Admin clicks "Time Ban" → Select duration (hours)
                                ↓
                        POST /admin/time_ban_user
                                ↓
                        Calculate ban_until (datetime + hours)
                                ↓
                        Update User: status = 'blocked'
                                ↓
                        Set ban_type = 'temporary'
                                ↓
                        Set ban_until timestamp
                                ↓
                        Commit to database
                                ↓
                        Socket.emit('user_banned')
                                ↓
                        User can't login until ban_until
                                ↓
                        Auto-unblock when time expires
```

### **User Unblock:**
```
Admin clicks "Unblock" → POST /admin/unblock_user
                                ↓
                        Update User: status = 'active'
                                ↓
                        Clear ban_until = NULL
                                ↓
                        Clear ban_type = NULL
                                ↓
                        Commit to database
                                ↓
                        Socket.emit('user_unblocked')
                                ↓
                        User can login again
```

### **Delete User:**
```
Admin clicks "Delete" → Confirm twice (safety)
                                ↓
                        POST /admin/delete_user
                                ↓
                        Find User record
                                ↓
                        CASCADE DELETE:
                                ↓
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
            Messages      GroupMembers     Sessions
                ↓               ↓               ↓
                └───────────────┼───────────────┘
                                ↓
                        Delete User record
                                ↓
                        Commit to database
                                ↓
                        Socket.emit('user_deleted')
                                ↓
                        User permanently removed
```

### **Change User Password:**
```
Admin clicks "Password" → Enter new password
                                ↓
                        POST /admin/change_user_password
                                ↓
                        Hash new password (werkzeug)
                                ↓
                        Update User record
                                ↓
                        Commit to database
                                ↓
                        User must use new password
```

---

##  6. System Architecture

### **Complete System Flow:**
```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT BROWSER                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  HTML/CSS   │  │ JavaScript  │  │ Socket.IO   │         │
│  │  Templates  │  │   Client    │  │   Client    │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│         │                 │                 │                 │
└─────────┼─────────────────┼─────────────────┼────────────────┘
          │                 │                 │
          ↓ HTTP            ↓ HTTP            ↓ WebSocket
          │                 │                 │
┌─────────┼─────────────────┼─────────────────┼────────────────┐
│         ↓                 ↓                 ↓                 │
│  ┌──────────────────────────────────────────────────┐       │
│  │           FLASK APPLICATION (app.py)              │       │
│  │                                                    │       │
│  │  ┌──────────────┐  ┌──────────────┐             │       │
│  │  │    Routes    │  │   SocketIO   │             │       │
│  │  │  /login      │  │   Handlers   │             │       │
│  │  │  /signup     │  │  - connect   │             │       │
│  │  │  /chat       │  │  - send_msg  │             │       │
│  │  │  /upload     │  │  - typing    │             │       │
│  │  │  /admin      │  │  - get_msgs  │             │       │
│  │  └──────┬───────┘  └──────┬───────┘             │       │
│  │         │                  │                      │       │
│  │         ↓                  ↓                      │       │
│  │  ┌──────────────────────────────────┐            │       │
│  │  │      SQLAlchemy ORM               │            │       │
│  │  └──────────────┬───────────────────┘            │       │
│  └─────────────────┼──────────────────────────────────┘       │
│                    │                                          │
└────────────────────┼──────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│                   SQLite DATABASE                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   User   │  │  Group   │  │ Message  │  │GroupMembr│  │
│  │          │  │          │  │          │  │          │  │
│  │ id       │  │ id       │  │ id       │  │ id       │  │
│  │ user_id  │  │ group_id │  │ msg_id   │  │ group_id │  │
│  │ username │  │ name     │  │ sender   │  │ username │  │
│  │ password │  │ creator  │  │ recipient│  │ joined_at│  │
│  │ email    │  │ avatar   │  │ message  │  │          │  │
│  │ role     │  │ created  │  │ msg_type │  │          │  │
│  │ status   │  │          │  │ file_url │  │          │  │
│  │ ban_info │  │          │  │ is_group │  │          │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└────────────────────────────────────────────────────────────┘
                     ↑
                     │
┌────────────────────┼──────────────────────────────────────┐
│                    │         FILE SYSTEM                   │
│                    │                                        │
│  static/uploads/   │                                        │
│    ├── profiles/   ←── User profile pictures              │
│    ├── images/     ←── Shared images                      │
│    ├── videos/     ←── Shared videos                      │
│    └── files/      ←── Documents, PDFs, etc.              │
└───────────────────────────────────────────────────────────┘
```

### **WebSocket Communication Flow:**
```
┌──────────────┐                    ┌──────────────┐
│   Client A   │                    │   Client B   │
│  (User 1)    │                    │  (User 2)    │
└──────┬───────┘                    └──────┬───────┘
       │                                    │
       │ Socket.emit('send_message')        │
       ├───────────────────────────────────►│
       │                                    │
       │          ┌──────────────┐         │
       │          │   Flask      │         │
       │          │   SocketIO   │         │
       │          │   Server     │         │
       │          └──────┬───────┘         │
       │                 │                  │
       │                 ↓                  │
       │         Save to Database          │
       │                 │                  │
       │                 ↓                  │
       │         Get user_sockets          │
       │                 │                  │
       │                 ↓                  │
       │    ┌────────────┴───────────┐     │
       │    ↓                        ↓     │
       │ emit to                  emit to  │
       │ sender                   recipient│
       │    │                        │     │
       ◄────┤                        ├────►│
       │    Socket.on('receive_message')  │
       │                                   │
       ↓                                   ↓
   Display                             Display
   Message                             Message
```

### **Online Status Tracking:**
```
User connects → Socket.on('connect')
                        ↓
                online_users[socket_id] = username
                        ↓
                user_sockets[username] = socket_id
                        ↓
                Socket.emit('user_connected', broadcast=True)
                        ↓
                All clients update online indicators
                        ↓
User disconnects → Socket.on('disconnect')
                        ↓
                Remove from online_users
                        ↓
                Remove from user_sockets
                        ↓
                Socket.emit('user_disconnected', broadcast=True)
                        ↓
                All clients hide online indicator
```

### **Profile Update Broadcast:**
```
User updates profile → POST /update_profile
                                ↓
                        Update database
                                ↓
                        Socket.emit('profile_updated')
                                ↓
                        Broadcast to all connected clients
                                ↓
                        All clients update user avatar/info
```

---

## 🔄 Data Flow Summary

### **Registration → Chat:**
```
Signup Form → Validate → Generate User ID → Hash Password 
    → Save to DB → Create Session → Redirect to Chat 
    → Load User List → Connect WebSocket → Ready to Chat
```

### **Send Message → Receive:**
```
Type Message → Upload File (optional) → Socket.emit 
    → Server Handler → Save to DB → Emit to Recipients 
    → Clients Receive → Display Message → Update UI
```

### **Admin Action → User Impact:**
```
Admin Action → Validate Admin Role → Update DB 
    → Socket Broadcast → User Affected → Can't Login (if banned) 
    → Or Data Deleted (if deleted)
```

---

## 📊 Database Relationships

```
User ─────┬──── Messages (sender)
          │
          ├──── Messages (recipient)
          │
          └──── GroupMember

Group ────┬──── GroupMember
          │
          └──── Messages

Message ──┬──── User (sender)
          │
          ├──── User (recipient)
          │
          └──── Group (optional)

GroupMember ─┬──── Group
              │
              └──── User (via username)
```

---

##  Technology Stack

```
┌─────────────────────────────────────────┐
│           FRONTEND                       │
│  - HTML5, CSS3                          │
│  - Vanilla JavaScript                   │
│  - Socket.IO Client                     │
└─────────────────────────────────────────┘
                  ↕
┌─────────────────────────────────────────┐
│           BACKEND                        │
│  - Flask (Python Web Framework)         │
│  - Flask-SocketIO (WebSocket)           │
│  - Flask-SQLAlchemy (ORM)               │
│  - Werkzeug (Security)                  │
└─────────────────────────────────────────┘
                  ↕
┌─────────────────────────────────────────┐
│           DATABASE                       │
│  - SQLite (speedtransfer.db)            │
└─────────────────────────────────────────┘
                  ↕
┌─────────────────────────────────────────┐
│        FILE STORAGE                      │
│  - Local Disk (static/uploads/)         │
└─────────────────────────────────────────┘
```

---

## Key Features Summary

✅ **Real-time messaging** with WebSocket  
✅ **File sharing** (images, videos, documents)  
✅ **Group chats** with multiple members  
✅ **User authentication** with sessions  
✅ **Admin panel** with user management  
✅ **Ban system** (permanent & temporary)  
✅ **Online status** tracking  
✅ **Typing indicators**  
✅ **Profile management**  
✅ **Password recovery** with User ID  

---

## Notes

- All passwords are **hashed** using Werkzeug
- File uploads are **limited to 50MB**
- User IDs are **auto-generated** (USR000001, USR000002, etc.)
- Admin account: `admin` / `admin123` (USR000001)
- WebSocket uses **room-based messaging** for efficiency
- Database uses **CASCADE DELETE** for data integrity
