# Background and Motivation

**Project**: AI Resume Refiner (Resume-Matcher fork)
**New Goal**: Deploy the application to a production Ubuntu server.
**URL**: The application will be accessible at `https://cvmatcher.best`.
curl ifconfig.me = 34.174.57.253
**Data Storage**: The SQLite database (`app.db`) will be stored and used on the Ubuntu server.
**Previous Status**: The application was successfully running in a local development environment on macOS. The focus is now on production deployment.

# Key Challenges and Analysis

Deploying the application with a dedicated domain simplifies the process significantly compared to using a subpath.

1.  **DNS Configuration**: The domain `cvmatcher.best` needs to be pointed to the server's public IP address by creating an 'A' record with your domain registrar.
2.  **Server Environment**: The Ubuntu server must have all necessary dependencies installed: Node.js, Python, `uv`, and a reverse proxy (Nginx is recommended).
3.  **Reverse Proxy (Virtual Host)**: Nginx will be configured to handle incoming requests for `cvmatcher.best`. It will serve the frontend application and forward API requests (e.g., `/api/*`) to the backend service. It will also be set up to handle SSL/TLS for `https` using Let's Encrypt.
4.  **Process Management**: The frontend and backend processes need to be managed as services (e.g., using `systemd`) to ensure they run continuously and restart on failure.
5.  **Environment Variables**: The `NEXT_PUBLIC_API_URL` on the frontend needs to be set to the production API URL, which will simply be `/api` because the reverse proxy will handle the routing on the same domain.
6.  **CORS**: The FastAPI backend's CORS policy must be updated to accept requests from `https://cvmatcher.best`.

# High-level Task Breakdown

Here is the simplified, step-by-step plan for deployment:

1.  **DNS Configuration**: Point the `cvmatcher.best` domain to the server's IP address.
    -   **Action**: Create an 'A' record for `@` and `www` pointing to the server's IP.
    -   **Success Criteria**: Running `dig cvmatcher.best +short` on any machine returns the server's correct IP address. (This can take some time to propagate).

2.  **Server Preparation**: Install all necessary software on the Ubuntu server.
    -   **Success Criteria**: `node --version`, `python --version`, `uv --version`, and `nginx -v` run successfully and show recent, compatible versions.

3.  **Code Deployment**: Transfer the application code to the server.
    -   **Success Criteria**: The project directory is copied to `/var/www/cvmatcher.best`.

4.  **Application Build on Server**: Install dependencies and build the application for production.
    -   **Success Criteria**: `npm install`, `uv sync`, and `npm run build` execute without errors on the server.

5.  **Application Services Setup**: Configure `systemd` to manage the backend and frontend processes.
    -   **Success Criteria**: `systemd` service files are created, and the services can be started, stopped, and enabled to run on boot.

6.  **Reverse Proxy & SSL Configuration**: Set up Nginx to serve the site over HTTPS.
    -   **Success Criteria**: Nginx configuration is created for `cvmatcher.best`. Running `certbot --nginx` successfully obtains and installs an SSL certificate. Requests to `https://cvmatcher.best` are correctly served by the frontend, and requests to `/api` are forwarded to the backend.

7.  **Final Verification**: Test the live application.
    -   **Success Criteria**: The application is fully functional at `https://cvmatcher.best`. All features, including resume uploads, work correctly.

# Project Status Board
- [ ] DNS Configuration
- [ ] Server Preparation
- [ ] Code Deployment
- [ ] Application Build on Server
- [ ] Application Services Setup
- [ ] Reverse Proxy & SSL Configuration
- [ ] Final Verification

# Executor's Feedback or Assistance Requests
*This section will be updated by the Executor during the implementation phase.*

# Lessons
*This section will be updated with any lessons learned during the deployment process.*
- **NEVER auto commit and push git changes before user approval. Always wait for explicit user permission before executing git commit and push commands.**
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- If there are vulnerabilities that appear in the terminal, run npm audit before proceeding
- Using a dedicated domain is much simpler than deploying to a subpath.
