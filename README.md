# Configure VPN for Private Apps and Show Lateral Movement Between 2 Private Apps with VPN Access

## Introduction
This project demonstrates the configuration of VPN for private applications and showcases lateral movement between two private apps with VPN access. The project includes the setup of two Docker containers (`app1` and `app2`), the deployment of an HTTP server, and a Python script with specific IP addresses and server details. Additionally, it compares traditional VPN solutions with Cloudflare Zero Trust, highlighting security features, performance, and user experience.

## Part 1: Traditional VPN Setup

### Step 1: Configuring the VPN on `app1` and `app2`

1. **Install OpenVPN**:
    ```bash
    sudo apt-get update
    sudo apt-get install openvpn
    ```

2. **Initialize OpenVPN Configuration for `app1`**:
    ```bash
    sudo docker run -v /etc/openvpn:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://<Server_IP>:1195
    sudo docker run -v /etc/openvpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
    ```

3. **Run OpenVPN Server in `app1`**:
    ```bash
    sudo docker run -v /etc/openvpn:/etc/openvpn -d -p 1195:1194/udp --cap-add=NET_ADMIN --name app1 kylemanna/openvpn
    ```

4. **Initialize OpenVPN Configuration for `app2`**:
    ```bash
    sudo docker run -v /etc/openvpn:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://<Server_IP>:1196
    sudo docker run -v /etc/openvpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
    ```

5. **Run OpenVPN Server in `app2`**:
    ```bash
    sudo docker run -v /etc/openvpn:/etc/openvpn -d -p 1196:1194/udp --cap-add=NET_ADMIN --name app2 kylemanna/openvpn
    ```

### Step 2: Demonstrating Lateral Movement between `app1` and `app2`

1. **Check Internal IP Address of `app1`**:
    ```bash
    sudo docker exec -it app1 ip addr show eth0 | grep inet
    ```

2. **Check Internal IP Address of `app2`**:
    ```bash
    sudo docker exec -it app2 ip addr show eth0 | grep inet
    ```

3. **Start Web Server on `app1`**:
    ```bash
    sudo docker exec -it app1 apk update
    sudo docker exec -it app1 apk add nginx
    sudo docker exec -it app1 nginx
    ```

4. **Verify Web Server on `app1`**:
    ```bash
    sudo docker exec -it app1 wget -qO- 172.17.0.2
    ```

5. **Access Web Server from `app2`**:
    ```bash
    sudo docker exec -it app2 wget -qO- 172.17.0.2:8081
    ```

### Demonstrating Restricted Access Without VPN

1. **Disable VPN**:
    - Ensure the VPN is not active in both `app1` and `app2`:
    ```bash
    sudo docker exec -it app1 pkill openvpn
    sudo docker exec -it app2 pkill openvpn
    ```

2. **Attempt to Access Web Server from `app2` Without VPN**:
    - Verify restricted access by attempting to connect to the web server on `app1` from `app2`:
    ```bash
    sudo docker exec -it app2 wget -qO- 172.17.0.2:8081
    ```

### Part 2: Cloudflare Zero Trust Setup

#### Step 1: Setting Up Cloudflare Zero Trust

1. **Create a Cloudflare Account**:
    - Go to [Cloudflare](https://www.cloudflare.com) and create an account if you don't already have one.

2. **Configure Cloudflare Zero Trust**:
    - Follow the Cloudflare Zero Trust documentation to set up your Zero Trust environment:
      - **Documentation Link**: [Cloudflare Zero Trust Docs](https://developers.cloudflare.com/cloudflare-one/)

3. **Deploying Zero Trust Policies**:
    - Create Zero Trust policies to control access to applications.

#### Step 2: Demonstrating Cloudflare Zero Trust

1. **Accessing Applications**:
    - Show how users connect to applications through Cloudflare Zero Trust, illustrating the security checks performed at each connection.

2. **Compare Security Features**:
    - Highlight security features such as identity verification, application-layer security, and conditional access.

## Conclusion

- **Summary**:
  - Recap the key points of comparison between traditional VPN and Cloudflare Zero Trust.
  - Highlight the benefits of Cloudflare Zero Trust in terms of security, performance, and user experience.

- **Q&A**:
  - Open the floor for any questions or additional demonstrations.

## Commands Cheat Sheet

### Traditional VPN Setup
```bash
# Install OpenVPN
sudo apt-get update
sudo apt-get install openvpn

# Initialize OpenVPN for `app1`
sudo docker run -v /etc/openvpn:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://<Server_IP>:1195
sudo docker run -v /etc/openvpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki

# Run OpenVPN Server for `app1`
sudo docker run -v /etc/openvpn:/etc/openvpn -d -p 1195:1194/udp --cap-add=NET_ADMIN --name app1 kylemanna/openvpn

# Initialize OpenVPN for `app2`
sudo docker run -v /etc/openvpn:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://<Server_IP>:1196
sudo docker run -v /etc/openvpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki

# Run OpenVPN Server for `app2`
sudo docker run -v /etc/openvpn:/etc/openvpn -d -p 1196:1194/udp --cap-add=NET_ADMIN --name app2 kylemanna/openvpn
