# Jenkins Build Setup

## 1. Download Jenkins
- Go to: [https://www.jenkins.io/download/](https://www.jenkins.io/download/)
- Download **Jenkins 2.528.2 LTS**

## 2. Run Jenkins
```bash
java -jar jenkins.war
```
- Open your browser: [http://localhost:8080](http://localhost:8080)
- Copy the initial admin password displayed in the CMD and paste it in Jenkins login.

## 3. Create Admin User
- Create your first admin user:
  - Username
  - Password
  - Email

## 4. Create New Jenkins Job
1. Click **New Item** (top-left)
2. Enter Job Name, e.g., `train-booking-service-build`
3. Select **Freestyle project**
4. Click **OK**

## 5. Configure Git Repository
- Select **Git**
- Enter your repository URL

## 6. Configure Build
- Scroll to **Build Steps**
- Add build step → **Invoke top-level Maven targets**
- **Install Maven locally**: [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)
- Add Maven `bin` path to environment variables
- Goals: `clean install`

## 7. Configure Maven in Jenkins
1. Go to **Manage Jenkins → Global Tool Configuration**
2. Scroll to **Maven → Add Maven**
3. Name: `Maven_3.9`
4. Maven Home: paste your local Maven path
5. Goals: `clean install`

## 8. Trigger Build
- Jobs → Configure → **Build periodically**
- Example cron:
```
0 9,21 * * *
```
- Runs every day at 9:00 AM and 9:00 PM

## 9. Setup Email Notification
### Gmail App Password
- Go to: [Google App Passwords](https://myaccount.google.com/apppasswords)
- App name: `Jenkins`
- Click **Create** → Copy the 16-character SMTP password

### Jenkins Email Configuration
- Manage Jenkins → **System → E-mail Notification**
  - SMTP server: `smtp.gmail.com`
  - Username: your Gmail
  - Password: app password
  - Use TLS ✔
  - Port: `587`

### Job Email Notifications
1. Job → Configure → Post-build Actions → **E-mail Notification**
2. Add your email(s)

### Extended Email Notifications
1. Post-build Actions → **Editable Email Notification**
2. Configure:
    - **Project Recipient List:** `yourgmail@gmail.com`
    - **Content Type:** HTML (to render your table properly)
    - **Default Subject:** `${PROJECT_NAME} - Build #${BUILD_NUMBER} - ${BUILD_STATUS}`
    - **Default Content:**
      ```html
      <!DOCTYPE html>
      <html>
      <body>
      <h2>Build Status: ${BUILD_STATUS}</h2>
 
      <h3>📌 Build Details</h3>
      <table border="1" cellpadding="6">
      <tr><td>Project</td><td>${PROJECT_NAME}</td></tr>
      <tr><td>Build Number</td><td>${BUILD_NUMBER}</td></tr>
      <tr><td>Date</td><td>${BUILD_TIMESTAMP}</td></tr>
      <tr><td>Triggered By</td><td>${CAUSE}</td></tr>
      <tr><td>Git Branch</td><td>${GIT_BRANCH}</td></tr>
      <tr><td>Commit</td><td>${GIT_COMMIT}</td></tr>
      </table>
      </body>
      </html>
      ```
    - **Attach Build Log** ✔ (optional)


### Email Triggers
- Scroll to **Add Trigger**
- Recommended:always

## 10. Custom HTML Email Templates
- Use **Editable Email Notification** for full customization
- Supports HTML content and build log attachments