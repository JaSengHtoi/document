
# 🚀 Speed Transfer - Real-time Chat Application

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![Flask](https://img.shields.io/badge/Flask-3.0+-green.svg)
![Socket.IO](https://img.shields.io/badge/Socket.IO-Real--time-orange.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)

A modern, feature-rich real-time chat application built with Flask and Socket.IO, featuring admin controls, group chats, file sharing, and comprehensive user management.

## ✨ Features

### 👥 User Features
- **Real-time Messaging**: Instant 1-on-1 and group chat with Socket.IO
- **File Sharing**: Upload and share images, videos, and documents (up to 50MB)
- **Custom User IDs**: Create and change your unique user identifier
- **Profile Customization**: Upload profile pictures and customize bio
- **Online Status**: See who's online in real-time
- **Read Receipts**: Know when messages are read
- **Typing Indicators**: See when others are typing
- **Group Chats**: Create and manage group conversations
- **Search Functionality**: Quickly find users and groups

### 👑 Admin Features
- **User Management Dashboard**: Comprehensive overview of all users
- **User Actions**:
  - Permanent block/unblock
  - Temporary time-based bans (1 hour to 1 month)
  - Force password reset
  - Account deletion
  - View detailed activity logs
- **Broadcasting**: Send messages to all users or specific individuals
- **Analytics**: Track user activity, device info, and chat statistics
- **Real-time Monitoring**: See online users and system stats

### 🔒 Security & Privacy
- Password hashing with Werkzeug
- Session management
- Activity logging with device detection
- IP address tracking
- Secure file uploads with validation

## 📋 Table of Contents

- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [API Documentation](#-api-documentation)
- [Database Schema](#-database-schema)
- [Screenshots](#-screenshots)
- [Contributing](#-contributing)
- [License](#-license)

## 🛠️ Installation

### Prerequisites

- Python 3.8 or higher
- pip (Python package manager)
- Git

### Step 1: Clone the Repository

```bash
git clone https://github.com/bawkli/speed-transfer.git
cd speed-transfer
```

### Step 2: Create Virtual Environment

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/Mac
python3 -m venv venv
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 4: Run the Application

```bash
python app.py
```

The application will start on `http://localhost:5000`

## 📦 Dependencies

```txt
Flask==3.0.0
flask-socketio==5.3.5
python-socketio==5.10.0
werkzeug==3.0.1
python-engineio==4.8.0
eventlet==0.33.3
Flask-SQLAlchemy==3.1.1
SQLAlchemy==2.0.23
```

## ⚙️ Configuration

### Default Settings

The application comes with pre-configured settings in `app.py`:

```python
# Database
SQLALCHEMY_DATABASE_URI = 'sqlite:///speedtransfer.db'

# File Upload
UPLOAD_FOLDER = 'static/uploads'
MAX_CONTENT_LENGTH = 50 * 1024 * 1024  # 50MB limit

# Allowed File Extensions
ALLOWED_EXTENSIONS = {
    'png', 'jpg', 'jpeg', 'gif',  # Images
    'mp4', 'avi', 'mov',           # Videos
    'pdf', 'doc', 'docx', 'txt', 'zip'  # Documents
}
```

### Admin Credentials

Default admin account (automatically created):
- **Username**: `admin`
- **Password**: `admin123`
- **User ID**: `ADMIN-SYS-ROOT`

⚠️ **Important**: Change the admin password after first login!

### Environment Variables (Optional)

Create a `.env` file for custom configuration:

```env
SECRET_KEY=your-secret-key-here
DATABASE_URI=sqlite:///speedtransfer.db
UPLOAD_FOLDER=static/uploads
MAX_FILE_SIZE=52428800
```

## 🚀 Usage

### For Regular Users

#### 1. Sign Up
1. Navigate to `/signup`
2. Fill in username, email, password
3. Optionally upload profile picture
4. Add bio (optional)
5. Click "Create Account"

#### 2. Login
- Use either **Username** or **User ID**
- Enter password
- Click "Login"

#### 3. Start Chatting
- Select a user from the sidebar
- Type your message
- Press Enter or click Send button
- Attach files using the 📎 button

#### 4. Create Groups
1. Click "Groups" tab
2. Click "Create New Group"
3. Enter group name
4. Select members
5. Click "Create Group"

#### 5. Settings
- Click ⚙️ icon in sidebar
- Update profile information
- Change User ID
- Change password
- Customize appearance
- Delete account (danger zone)

### For Administrators

#### 1. Access Admin Dashboard
- Login with admin credentials
- Automatically redirected to `/admin`

#### 2. User Management
- **View All Users**: See comprehensive user list with stats
- **Block User**: Permanently block access
- **Time Ban**: Temporarily ban (1 hour to 1 month)
- **Change Password**: Reset user passwords
- **Delete User**: Permanently remove account
- **View Activity**: See detailed user activity logs

#### 3. Broadcasting
1. Go to "Send Messages" tab
2. Select "BROADCAST TO ALL USERS" or specific user
3. Type message
4. Click "Send Message"

#### 4. Monitor Activity
- Click 📊 Activity button on any user
- View login/logout history
- See device information
- Check chat statistics

## 📡 API Documentation

### Authentication Endpoints

#### POST `/login`
Login user with username/user_id and password

**Request Body:**
```json
{
  "username": "bawkli",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "redirect": "/chat"
}
```

#### POST `/signup`
Register new user account

**Request Body (Form Data):**
```
username: bawkli 
email: bawkli@example.com
password: password123
bio: Hello World
profile_pic: [File]
```

**Response:**
```json
{
  "success": true
}
```

#### POST `/forgot_password`
Reset password using User ID

**Request Body:**
```json
{
  "user_id": "USR-20241119-ABC123",
  "new_password": "newpassword123"
}
```

### User Endpoints

#### POST `/update_profile`
Update user profile information

**Request Body (Form Data):**
```
email: newemail@example.com
bio: Updated bio
profile_pic: [File]
```

#### POST `/change_user_id`
Change user's unique identifier

**Request Body:**
```json
{
  "new_user_id": "MY-CUSTOM-ID"
}
```

#### POST `/change_password`
Change user password

**Request Body:**
```json
{
  "old_password": "oldpass123",
  "new_password": "newpass123"
}
```

#### POST `/delete_account`
Permanently delete user account

### Admin Endpoints

#### POST `/admin/block_user`
Permanently block a user

**Request Body:**
```json
{
  "username": "bawkli"
}
```

#### POST `/admin/unblock_user`
Unblock a user

**Request Body:**
```json
{
  "username": "bawkli"
}
```

#### POST `/admin/time_ban_user`
Temporarily ban user

**Request Body:**
```json
{
  "username": "bawkli",
  "hours": 24
}
```

#### POST `/admin/delete_user`
Delete user account (admin)

**Request Body:**
```json
{
  "username": "bawkli"
}
```

#### POST `/admin/change_user_password`
Reset user password (admin)

**Request Body:**
```json
{
  "username": "bawkli",
  "new_password": "newpass123"
}
```

#### POST `/admin/send_message`
Send message as admin

**Request Body:**
```json
{
  "recipient": "all",
  "message": "System maintenance at 10 PM"
}
```

#### GET `/admin/user_activity/<username>`
Get detailed user activity logs

**Response:**
```json
{
  "success": true,
  "user": {
    "username": "bawkli",
    "user_id": "USR-20251119-ABC123",
    "messages_sent": 150,
    "messages_received": 200,
    "created_at": "2025-11-19 10:30 AM",
    "last_seen": "2025-11-19 05:45 PM"
  },
  "activities": [...],
  "chat_stats": [...]
}
```

### Chat & Groups

#### POST `/create_group`
Create new chat group

**Request Body:**
```json
{
  "name": "Project Team",
  "members": ["alice", "bob", "charlie"]
}
```

#### POST `/upload`
Upload file for sharing

**Request Body (Form Data):**
```
file: [File]
```

**Response:**
```json
{
  "success": true,
  "file_url": "/static/uploads/images/20241119_153045_photo.jpg",
  "file_type": "images"
}
```

### WebSocket Events

#### Client → Server

**`send_message`**
```javascript
socket.emit('send_message', {
  recipient: 'bawkli',
  message: 'Hello!',
  type: 'text',
  file_url: '',
  is_group: false
});
```

**`get_messages`**
```javascript
socket.emit('get_messages', {
  recipient: 'bawkli',
  is_group: false
});
```

**`typing`**
```javascript
socket.emit('typing', {
  recipient: 'bawkli',
  is_group: false
});
```

**`delete_message`**
```javascript
socket.emit('delete_message', {
  message_id: 'msg-uuid-123'
});
```

#### Server → Client

**`receive_message`**
```javascript
socket.on('receive_message', (data) => {
  // data contains: id, sender, message, type, timestamp, etc.
});
```

**`load_messages`**
```javascript
socket.on('load_messages', (data) => {
  // data.messages contains array of messages
});
```

**`user_connected`**
```javascript
socket.on('user_connected', (data) => {
  // data.online_users contains list of online users
});
```

**`user_disconnected`**
```javascript
socket.on('user_disconnected', (data) => {
  // User went offline
});
```

**`user_typing`**
```javascript
socket.on('user_typing', (data) => {
  // Show typing indicator
});
```

**`force_logout`**
```javascript
socket.on('force_logout', (data) => {
  // Admin forced logout
  alert(data.reason);
  window.location.href = '/logout';
});
```

**`admin_broadcast`**
```javascript
socket.on('admin_broadcast', (data) => {
  // Show admin broadcast notification
});
```

## 🗄️ Database Schema

### User Table
```sql
CREATE TABLE user (
    id INTEGER PRIMARY KEY,
    user_id VARCHAR(50) UNIQUE NOT NULL,
    username VARCHAR(80) UNIQUE NOT NULL,
    email VARCHAR(120) NOT NULL,
    password VARCHAR(200) NOT NULL,
    role VARCHAR(20) DEFAULT 'user',
    profile_pic VARCHAR(200) DEFAULT '',
    bio TEXT DEFAULT '',
    status VARCHAR(20) DEFAULT 'active',
    ban_until DATETIME,
    ban_type VARCHAR(20),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_seen DATETIME
);
```

### Message Table
```sql
CREATE TABLE message (
    id INTEGER PRIMARY KEY,
    message_id VARCHAR(50) UNIQUE NOT NULL,
    sender_id INTEGER REFERENCES user(id),
    recipient_id INTEGER REFERENCES user(id),
    group_id INTEGER REFERENCES group(id),
    message TEXT DEFAULT '',
    msg_type VARCHAR(20) DEFAULT 'text',
    file_url VARCHAR(300) DEFAULT '',
    is_group BOOLEAN DEFAULT FALSE,
    is_read BOOLEAN DEFAULT FALSE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Group Table
```sql
CREATE TABLE group (
    id INTEGER PRIMARY KEY,
    group_id VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    creator VARCHAR(80) NOT NULL,
    avatar VARCHAR(200) DEFAULT '',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### GroupMember Table
```sql
CREATE TABLE group_member (
    id INTEGER PRIMARY KEY,
    group_id INTEGER REFERENCES group(id),
    username VARCHAR(80) NOT NULL,
    joined_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### ActivityLog Table
```sql
CREATE TABLE activity_log (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES user(id),
    activity_type VARCHAR(50) NOT NULL,
    details TEXT DEFAULT '',
    device_info VARCHAR(200) DEFAULT '',
    ip_address VARCHAR(50) DEFAULT '',
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

## 🎨 Project Structure

```
speed-transfer/
├── app.py                      # Main Flask application
├── debug_chat.py              # Debug utility script
├── requirements.txt           # Python dependencies
├── README.md                  # This file
│
├── instance/
│   └── speedtransfer.db      # SQLite database
│
├── static/
│   └── uploads/              # User uploaded files
│       ├── images/           # Image files
│       ├── videos/           # Video files
│       ├── files/            # Documents
│       └── profiles/         # Profile pictures
│
└── templates/                # HTML templates
    ├── login.html           # Login page
    ├── signup.html          # Registration page
    ├── forgot_password.html # Password reset
    ├── chat.html            # Main chat interface
    ├── admin.html           # Admin dashboard
    └── settings.html        # User settings
```

## 🔧 Debugging

### Debug Script

Use the included debug script to check chat visibility:

```bash
python debug_chat.py <username>
```

Example:
```bash
python debug_chat.py alice
```

This will show:
- What users Alice can see
- User details and roles
- Possible configuration issues

### Common Issues

#### Issue: Users can't see each other

**Solution**: Check if:
1. Both users have `role='user'` (not admin)
2. Users are in `active` status
3. Multiple users exist in database

#### Issue: Files not uploading

**Solution**: 
1. Check upload folder permissions
2. Verify file size (max 50MB)
3. Ensure file extension is allowed

#### Issue: Socket.IO not connecting

**Solution**:
1. Check if port 5000 is available
2. Verify CORS settings
3. Check browser console for errors

## 🎯 User ID System

### Format
```
USR-YYYYMMDD-XXXXXXXX
```

Example: `USR-20241119-A3F9K2L7`

### Features
- Automatically generated on signup
- Date-stamped (YYYYMMDD)
- 8 random alphanumeric characters
- User-changeable (8-30 characters)
- Validation: Letters, numbers, dash, underscore only

### Changing User ID

Users can change their ID with restrictions:
- 8-30 characters
- Only `A-Z`, `a-z`, `0-9`, `-`, `_`
- Must be unique across system

## 📱 Device Detection

The system automatically detects:
- 📱 iPhone
- 📱 Android
- 📱 Mobile (generic)
- 📱 Tablet/iPad
- 💻 Windows
- 💻 Mac
- 💻 Linux
- 💻 Desktop (generic)

## 🔐 Security Best Practices

### For Deployment

1. **Change Default Admin Password**
```python
# First login, then change password in settings
```

2. **Use Environment Variables**
```python
import os
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
```

3. **Enable HTTPS**
```python
socketio.run(app, ssl_context='adhoc')
```

4. **Database Security**
```bash
# Set proper permissions
chmod 600 instance/speedtransfer.db
```

5. **Limit Upload Size**
```python
app.config['MAX_CONTENT_LENGTH'] = 50 * 1024 * 1024
```

## 🌐 Deployment

### Production with Gunicorn

```bash
pip install gunicorn

gunicorn --worker-class eventlet -w 1 -b 0.0.0.0:5000 app:app
```

### Docker Deployment

Create `Dockerfile`:
```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000
CMD ["python", "app.py"]
```

Build and run:
```bash
docker build -t speed-transfer .
docker run -p 5000:5000 speed-transfer
```

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

    location /socket.io {
        proxy_pass http://localhost:5000/socket.io;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## 🧪 Testing

### Manual Testing Checklist

- [ ] User registration
- [ ] Login with username
- [ ] Login with User ID
- [ ] Password reset
- [ ] Profile update
- [ ] User ID change
- [ ] Send text message
- [ ] Upload image
- [ ] Upload video
- [ ] Upload document
- [ ] Create group
- [ ] Group messaging
- [ ] Admin block user
- [ ] Admin time ban
- [ ] Admin broadcast
- [ ] Activity logging

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

### Code Style

- Follow PEP 8 for Python code
- Use meaningful variable names
- Add comments for complex logic
- Update documentation for new features

## 📝 License

This project is licensed under the MIT License - see below for details:

```
MIT License

Speed Transfer

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 👨‍💻 Author

**Speed Transfer Team**
- GitHub: [@bawkli](https://github.com/bawkli)
- Email: naw170186@gmail.com

## 🙏 Acknowledgments

- Flask framework and community
- Socket.IO for real-time communication
- SQLAlchemy for database management
- All contributors and testers

## 📞 Support

For support and questions:
- 📧 Email: naw170186@gmail.com
- 🐛 Issues: [GitHub Issues](https://github.com/bawkli/speed-transfer/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/bawkli/speed-transfer/discussions)


### Version !2 (Planned)
- [ ] Voice messages
- [ ] Video calls
- [ ] Message reactions
- [ ] Message forwarding
- [ ] Dark mode
- [ ] Mobile app (React Native)
- [ ] End-to-end encryption
- [ ] Message search
- [ ] User status messages
- [ ] Pinned messages

### Version !3 (Future)
- [ ] Story/Status feature
- [ ] Voice/Video rooms
- [ ] Bot API
- [ ] Webhook integration
- [ ] Advanced analytics
- [ ] Multi-language support

---

⭐ **If you find this project helpful, please give it a star!** ⭐

Made with ❤️ by Speed Transfer Team
