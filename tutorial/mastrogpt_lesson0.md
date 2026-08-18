# Welcome to the Course

Hello everyone, and welcome to the course **Application Development with LLM Open using Apache Open Serverless**.  
It’s quite a long title (worthy of Wertmüller!), but for friends, this is simply the **MastroGPT course**.

---

## 0. Getting Started

Before diving into the lessons, let’s start by setting up the environment.  
I have already started mine, but here’s how you can do it too:

1. Go to [MastroGPT on GitHub](https://github.com/mastrogpt).  
2. Open the **starter project**.  
3. Click the **Code** button and choose **Create Codespace on main**.  

This will launch your environment in the cloud. It may take a while the first time, which is why I prepared mine in advance.

⚠️ You are *not obliged* to use Codespaces. If you prefer, you can work locally with Docker or another setup.  
However, Codespaces is convenient and saves a lot of configuration time, so I recommend it.

---

## 1. The Extension

Once the environment is ready, you’ll see some icons on the left sidebar:

- ☁️ **Cloud Icon** → starts the main extension. The first button is **Login**.  
  - Enter your username and password.  
  - If successful, you’ll see: *“You are now logged in to Open Serverless.”*  

- 🧪 **Test Tube Icon** → runs the tests included with the environment.  

- 📄 **Docs Icon** → shows the documentation and lessons.  
  - You can view both **source** (Markdown) and **rendered slides**.  
  - Having access to the source is useful for exercises.

---

## 2. First Deployment & Testing

1. Click the **Deploy** button.  
   - This installs the initial code into your environment.  

2. Once deployed, you can run the **tests**.  
   - If they pass, everything is working correctly.  

3. For development work, switch to **Development Mode (dev)**.  
   - This launches a local web server for the UI.  
   - Look for the 🌐 **Globe Icon** (sometimes hidden under the “antenna” section) to open it in the browser.

---

## 3. Meet *Pinocchio* (The UI)

The web interface is called **Pinocchio** (a playful reference to Mastro Geppetto).  

- Default credentials:  
  - **User**: Pinocchio  
  - **Password**: Geppetto  

You should change the password immediately. Here’s how:

```bash
# Change password for Pinocchio
ops ai user update Pinocchio NEW_PASSWORD

# Redeploy login service
ops upsert deploy mastrogpt-login
```

The interface supports:

- Multiple chats (e.g., *Hello Chat* simply replies “hello”).  
- Rich outputs: code blocks, HTML, chessboards, forms, etc.  
- File upload for exercises.  
- A customizable **side view** (open/close as needed).

---

## 4. Using the Terminal

Much of the course involves the command line.  

- Open a terminal in Codespaces:  
  - Menu → **Terminal > New Terminal**  

Example: Change the default password using CLI commands (see above).  

---

## 5. Local Setup Options

If you don’t want to use Codespaces:  

- Install **Nuvolaris** locally with Docker.  
- Or deploy on providers like AWS, GCP, Azure, Akamai, or Hetzner.  
- Nuvolaris supports multiple databases and services (Redis, Minio, PostgreSQL, etc.).

We can dedicate an ad-hoc session to installation methods if needed.

---

## 6. Course Support

For help and discussion:  

- 💬 **Discord** → our main support channel (with an Italian channel too).  
- 📢 **Reddit** → for community discussions.  
- 🌐 [MastroGPT.com](https://mastrogpt.com) → request an account via chatbot.  
- 📧 LinkedIn → you can also reach us directly.  

---

## 7. Next Step

This was **Lesson 0 (Pre-lesson)** — setup and orientation.  
Now, go to the extension → **Lessons** → **First Lesson** to continue.

---

✅ You’re ready to begin the real journey into Application Development with LLM Open!  
