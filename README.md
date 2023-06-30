#!/usr/bin/env python3

import sys
import subprocess

def install_docker():
    subprocess.run(['curl', '-fsSL', 'https://get.docker.com', '-o', 'get-docker.sh'])
    subprocess.run(['sudo', 'sh', 'get-docker.sh'])
    subprocess.run(['sudo', 'usermod', '-aG', 'docker', '$USER'])
    subprocess.run(['sudo', 'systemctl', 'enable', 'docker'])
    subprocess.run(['sudo', 'systemctl', 'start', 'docker'])

def install_docker_compose():
    subprocess.run(['sudo', 'curl', '-L', 'https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)', '-o', '/usr/local/bin/docker-compose'])
    subprocess.run(['sudo', 'chmod', '+x', '/usr/local/bin/docker-compose'])

def check_dependencies():
    try:
        subprocess.run(['docker', '--version'], stdout=subprocess.PIPE, check=True)
        subprocess.run(['docker-compose', '--version'], stdout=subprocess.PIPE, check=True)
    except FileNotFoundError:
        print("Docker or Docker Compose is not installed.")
        choice = input("Do you want to install Docker and Docker Compose? (Y/N): ").lower()
        if choice == 'y':
            install_docker()
            install_docker_compose()
            print("Docker and Docker Compose installed successfully.")
        else:
            sys.exit(1)

def create_wordpress_site(site_name):
    subprocess.run(['sudo', 'docker-compose', 'up', '-d'])

def add_hosts_entry(site_name):
    with open('/etc/hosts', 'a') as hosts_file:
        hosts_file.write(f'127.0.0.1 {site_name}\n')

def open_in_browser(site_name):
    print(f"Open http://{site_name} in your browser.")

def enable_disable_site(action):
    if action == 'start':
        subprocess.run(['sudo', 'docker-compose', 'start'])
    elif action == 'stop':
        subprocess.run(['sudo', 'docker-compose', 'stop'])

def delete_site():
    subprocess.run(['sudo', 'docker-compose', 'down'])
    # Additional code to delete local files if any

def main():
    if len(sys.argv) < 2:
        print("Usage: python script.py [create|enable|disable|delete] [site_name]")
        sys.exit(1)

    command = sys.argv[1]
    site_name = sys.argv[2] if len(sys.argv) >= 3 else ''

    if command == 'create':
        check_dependencies()
        create_wordpress_site(site_name)
        add_hosts_entry(site_name)
        open_in_browser(site_name)
    elif command == 'enable' or command == 'disable':
        enable_disable_site(command)
    elif command == 'delete':
        delete_site()
    else:
        print("Invalid command.")

if __name__ == '__main__':
    main()
