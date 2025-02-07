# Self-Hosting Calendar with Baikal and Morgen

Privacy and security are often overlooked when choosing productivity tools, even though these platforms holds some of your most sensitive information. For instance, A calendar application can reveal details about your daily routine, appointments, travel plans, confidential internal projects and some events attached with critical information as notes.

If you want complete ownership of your data to safeguard personal information or business secrets, self-hosting a calendar application is a good solution.

To get started with self-hosting a CalDAV server, you'll need Baikal (a lightweight CalDAV server) and Docker to run it locally. You can find Docker installation instructions on the official [Docker documentation](https://docs.docker.com/engine/install) and explore [Docker Desktop](https://docs.docker.com/desktop) for a user-friendly GUI option.


## What is CalDAV?

**CalDAV** is a standardized protocol to manage and synchronize calendar events across multiple devices. CalDAV works as a client-server architecture between a calendar client (like Morgen, Thunderbird, or Apple Calendar) and a calendar server (like Baikal). The server stores calendar data, while the client syncs with the server to fetch updates and manage appointments. When you add, delete, or update events on a CalDAV-compatible client like Morgen, these changes are sent to the server and reflected across all synced devices.

We'll use Baikal, a lightweight open-source CalDAV and CardDAV server. However, if you or your organization already self-host services like Nextcloud or ownCloud, check whether they have built-in CalDAV support. These platforms often include this feature, eliminating the need for an extra service like Baikal.

## Setting Up a CalDAV Server

Setting up a CalDAV server might sound complicated, but trust me it's easier than you think, especially for personal use cases where you can run it locally using Docker. But first, double-check that Docker is installed on your system.

We'll use the lightweight Nginx variant instead of Apache since it's less than half the size. Lean and efficient, just how we like it.

#### Start Baikal

```bash
docker run --rm -it -p 80:80 ckulka/baikal:nginx
```

To quickly test Baikal, run this Docker command. However, for a more structured and scalable setup (especially if you want better CI/CD practices), we recommend using `docker-compose.yml`.

### Step-by-Step Guide to Setup Baikal

**Step 1:** Create a directory to hold your Baïkal configurations, and it's a good idea to back it up if you ever move infrastructure.

```bash
sudo mkdir /opt/baikal && sudo chown $USER:$USER /opt/baikal
```

This command creates a directory under `/opt/baikal` and changes ownership to your current user to avoid needing `sudo` later.

**Step 2:** Create a *docker-compose.yml* file under `/opt/baikal`.

```yml
services:
  baikal:
    image: ckulka/baikal:nginx
    restart: always # restart on reboot to avoid down time
    ports:
      - 80:80       # expose baikal on localhost port 80
    volumes:
      - /opt/baikal/config:/var/www/baikal/config         # store baikal configrations
      - /opt/baikal/Specific:/var/www/baikal/Specific     # store baikal configrations
```

**Step 3:** Once you start Baikal. Visit `http://127.0.0.1` in your browser. You’ll be greeted by a setup page where you can select your time zone and create an admin account.

```bash
cd /opt/baikal && docker compose up -d
```

![img](./attachments/f1.png)

**Step 4:** Choose a Database. Baikal supports SQLite, MySQL, and PostgreSQL. For most personal or small-scale use cases, SQLite works perfectly. Select it, save, and continue.

**Step 5:** Use your admin credentials to access the main dashboard. From here, you can tweak any configuration settings we made so far.

**Step 6:** To start using calendar you need an user and it can't be the admin account be just created. Head to the *"Users and Resources"* section to set up your first user. No need for an actual email address.

**Step 7:** After creating a user click on *Calendars*. You can create multiple calendars per user, similar to Google Calendar. By default a calendar is automatically created named as *"Default Calendar"*.

![img](./attachments/f2.png)

**Step 8:** Click *Edit*, enable *Notes*, and copy the CalDAV URI by clicking the info (`i`) button. Be sure to only copy path up to `/dav.php` to avoid permission issues (e.g., `http://127.0.0.1/dav.php`).

![img](./attachments/f3.png)

That's it! just use new user credentials and URI to integrate with your Morgen calendar.

Though steps are fairly easy to follow and similar to most self-hosting process. However, if you feel stuck, I've included a step-by-step YouTube video below for your convenience.

## Integrate Baikal with Morgen Calendar

Connecting Baikal to Morgen via CalDAV is quick and easy. Depending on your Morgen configuration, follow the appropriate path. Once you hit the CalDAV option, just enter your username, password, and Baikal server URL.

- **First Time Adding a Calendar:** Go to Settings > Profile > Calendars > Add an account > Other (CalDAV).
- **Premium Users (Adding Another Calendar):** Try Settings > Profile > Calendars > Add account > Calendar accounts > Other (CalDAV).
- **Alternate Method:** Settings > Profile > External accounts > Add account > Connect a calendar account > Other (CalDAV).

**Pro Tip:** Morgen supports all major calendar providers. But if your preferred provider isn't listed, check whether it offers CalDAV support. If so, you can still seamlessly integrate it with Morgen using the same steps.

![img](./attachments/output.gif)

### What Morgen Does to Protect Your Privacy?

At Morgen, your privacy is our priority. We don't sell your data or share it with third parties. Any information we do collect is securely stored and processed on cloud infrastructure that complies with GDPR regulations. Our data centers are located in Switzerland and the European Union, both regions are known for their strict data protection laws.

It's important to note that Morgen is solely a calendar client. Your events remain stored on the servers of your selected calendar provider (such as Google or Outlook) and are subject to their privacy policies.

To address this concern, Morgen offers CalDAV integration, giving you the ability to self-host your calendar and maintain full control over your schedule and organizational data. This way, your life's routines and sensitive information stay truly yours.

## What's Next?

- Enable HTTPS to encrypt data exchanges
	- Route Baikal traffic through a reverse proxy like Traefik or Caddy.
- Set up regular backups to S3 or another cloud storage service.
- Add an SMTP server to send and receive emails, ideal for organizational use.
- Check out the [Baikal Docker repository](https://github.com/ckulka/baikal-docker) for more advanced configuration options and resources.