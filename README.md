[![Build Stable](https://github.com/frappe/frappe_docker/actions/workflows/build_stable.yml/badge.svg)](https://github.com/frappe/frappe_docker/actions/workflows/build_stable.yml)
[![Build Develop](https://github.com/frappe/frappe_docker/actions/workflows/build_develop.yml/badge.svg)](https://github.com/frappe/frappe_docker/actions/workflows/build_develop.yml)

Everything about [Frappe](https://github.com/frappe/frappe) and [ERPNext](https://github.com/frappe/erpnext) in containers.

# Getting Started

**New to Frappe Docker?** Read the [Getting Started Guide](docs/getting-started.md) for a comprehensive overview of repository structure, development workflow, custom apps, Docker concepts, and quick start examples.

To get started you need [Docker](https://docs.docker.com/get-docker/), [docker-compose](https://docs.docker.com/compose/), and [git](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git) setup on your machine. For Docker basics and best practices refer to Docker's [documentation](http://docs.docker.com).

Once completed, chose one of the following two sections for next steps.

### Try in Play With Docker

To play in an already set up sandbox, in your browser, click the button below:

<a href="https://labs.play-with-docker.com/?stack=https://raw.githubusercontent.com/frappe/frappe_docker/main/pwd.yml">
  <img src="https://raw.githubusercontent.com/play-with-docker/stacks/master/assets/images/button.png" alt="Try in PWD"/>
</a>

### Try on your Dev environment

First clone the repo:

```sh
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

Then run: `docker compose -f pwd.yml up -d`

### To run on ARM64 architecture follow this instructions

After you clone the repo and `cd frappe_docker`, run this command to build multi-architecture images specifically for ARM64.

`docker buildx bake --no-cache --set "*.platform=linux/arm64"`

and then

- add `platform: linux/arm64` to all services in the `pwd.yml`
- replace the current specified versions of erpnext image on `pwd.yml` with `:latest`

Then run: `docker compose -f pwd.yml up -d`

## Final steps

Wait for 5 minutes for ERPNext site to be created or check `create-site` container logs before opening browser on port 8080. (username: `Administrator`, password: `admin`)

If you ran in a Dev Docker environment, to view container logs: `docker compose -f pwd.yml logs -f create-site`. Don't worry about some of the initial error messages, some services take a while to become ready, and then they go away.

# Making Changes to ERPNext

To make changes to ERPNext code, you need to set up a development environment. Here's how:

## Quick Setup for Development

1. **Set up Dev Container:**
   
   **On Windows (PowerShell):**
   ```powershell
   Copy-Item -Path devcontainer-example -Destination .devcontainer -Recurse
   ```
   
   **On Windows (Git Bash or WSL):**
   ```sh
   cp -R devcontainer-example .devcontainer
   ```
   
   **On Linux/Mac:**
   ```sh
   cp -R devcontainer-example .devcontainer
   ```
   
   This copies the devcontainer configuration files to `.devcontainer` directory, which VSCode will use to set up your development container.
   
   **Verify it worked:** You should now see a `.devcontainer` folder in your project root with `devcontainer.json` and `docker-compose.yml` files inside.

2. **Open in VSCode with Dev Containers:**
   - Install the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) in VSCode
   - Open the `frappe_docker` folder in VSCode
   - Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac) and select `Dev Containers: Reopen in Container`
   - Wait for the container to build (first time takes ~5 minutes)

3. **Inside the container, set up your bench:**
   ```sh
   cd /workspace/development
   python installer.py
   ```
   Follow the prompts to create a site and install ERPNext.

4. **Enable Developer Mode:**
   ```sh
   cd frappe-bench
   bench --site development.localhost set-config developer_mode 1
   bench --site development.localhost clear-cache
   ```
   > **Note:** The `installer.py` script automatically sets developer mode globally, but it's good practice to ensure it's enabled for your site as well.

5. **Start the development server:**
   ```sh
   bench start
   ```

## Where to Find ERPNext Code

Once set up, ERPNext code is located at:
```
development/frappe-bench/apps/erpnext/
```

You can edit files directly in this directory. Common locations:
- **DocTypes:** `apps/erpnext/erpnext/[module]/doctype/[doctype_name]/`
- **Python files:** `apps/erpnext/erpnext/[module]/`
- **JavaScript/Client scripts:** `apps/erpnext/erpnext/[module]/doctype/[doctype_name]/[doctype_name].js`
- **Frontend/UI:** `apps/erpnext/erpnext/public/js/` or in DocType folders
- **Logo/Images:** `apps/erpnext/erpnext/public/images/` or `apps/frappe/frappe/public/images/`

## Making Changes

### Creating a Git Branch for ERPNext Changes

Since ERPNext is installed as a git repository, you can create a branch to track your changes:

```bash
cd /workspace/development/frappe-bench/apps/erpnext
git checkout -b my-custom-changes
```

This creates a new branch called `my-custom-changes` where you can make your modifications.

### Making and Testing Changes

1. **Edit files directly** in `development/frappe-bench/apps/erpnext/`
2. **Rebuild assets** after making frontend changes:
   ```sh
   cd /workspace/development/frappe-bench
   bench build --app erpnext
   ```
3. **Clear cache** if changes don't appear:
   ```sh
   bench --site development.localhost clear-cache
   ```
4. **Restart bench** if needed (stop with `Ctrl+C` and run `bench start` again)

### Troubleshooting: Changes Not Showing

If your changes (logo, text, etc.) are not appearing, try these steps in order:

1. **Clear browser cache:**
   - Press `Ctrl+Shift+R` (Windows) or `Cmd+Shift+R` (Mac) for hard refresh
   - Or open browser in incognito/private mode

2. **Clear Frappe cache:**
   ```bash
   cd /workspace/development/frappe-bench
   bench --site development.localhost clear-cache
   ```

3. **Rebuild assets:**
   ```bash
   bench build --app erpnext
   # Or rebuild all apps
   bench build
   ```

4. **Restart bench server:**
   ```bash
   # Stop bench (Ctrl+C in the terminal running bench start)
   # Then restart:
   bench start
   ```

5. **Check developer mode is enabled:**
   ```bash
   bench --site development.localhost set-config developer_mode 1
   bench --site development.localhost clear-cache
   ```

6. **For logo changes specifically:**
   - Make sure you replaced the correct file
   - Check file permissions: `ls -la apps/erpnext/erpnext/public/images/`
   - Verify the file was saved correctly
   - Try clearing browser cache completely

7. **For text changes:**
   - If changing translations, clear cache and rebuild
   - If changing hardcoded text in Python/JS files, restart bench
   - Check if you're editing the right file (use browser DevTools to find the source)
   - **For JSON files (like home.json):** Clear cache and restart bench
   - **For Python files:** Restart bench (Python auto-reloads, but sometimes needs restart)
   - **For JavaScript files:** Rebuild assets with `bench build --app erpnext`

### Committing Your Changes

#### For frappe_docker Repository (Docker Config)

```bash
# In your frappe_docker directory
git add .
git commit -m "Description of your changes"
git push origin <your-branch-name>
```

#### For ERPNext Repository (App Code)

```bash
# Inside the container
cd /workspace/development/frappe-bench/apps/erpnext

# Check what changed
git status

# Create a branch if not already on one
git checkout -b my-custom-changes

# Add and commit changes
git add .
git commit -m "Description of your changes"

# Push to your GitHub fork
git push origin my-custom-changes
```

**Note:** Workspace database changes (like `is_hidden` flag) are not in git files. To persist them:
1. Export workspaces as JSON: `bench --site <site> export-doc Workspace "Workspace Name"`
2. Commit the exported JSON files
3. Or create a migration script that applies changes on deployment

## Resuming Work After Shutdown

### Option 1: Using VSCode Dev Container (Recommended)

1. **Open VSCode** in your project folder (`F:\villaex projects\frappe_docker`)
2. **VSCode will automatically detect** the dev container and ask to reopen
3. **Click "Reopen in Container"** when prompted
4. **Wait for container to start** (~30 seconds)
5. **Start the development server:**
   ```bash
   cd /workspace/development/frappe-bench
   bench start
   ```

### Option 2: Manual Container Start

If VSCode doesn't auto-detect:

1. **Open VSCode** in your project folder
2. **Press `Ctrl+Shift+P`** and type "Dev Containers: Reopen in Container"
3. **Select it** and wait for container to start
4. **Start bench:**
   ```bash
   cd /workspace/development/frappe-bench
   bench start
   ```

### Option 3: Using Docker Commands (If VSCode doesn't work)

```bash
# Start the dev container services
cd "F:\villaex projects\frappe_docker"
docker compose -f .devcontainer/docker-compose.yml up -d

# Enter the container
docker exec -it devcontainer-frappe-1 bash

# Inside container, start bench
cd /workspace/development/frappe-bench
bench start
```

### Important Notes

- **Your changes are saved** - All files in `development/` are on your local machine, so they persist after shutdown
- **Docker containers stop** when you shut down, but they restart automatically when you reopen VSCode
- **Database data persists** - MariaDB data is stored in a Docker volume, so your data remains
- **Git branches persist** - Your git branches and commits are saved in the `apps/erpnext/` folder

## Important Notes

- **Developer Mode** must be enabled for changes to take effect
- **Frontend changes** require running `bench build` to compile assets
- **Python changes** are hot-reloaded automatically when `bench start` is running
- The `development/` directory is git-ignored in the main repo, but ERPNext has its own git repository
- **Always work on a branch** - Don't modify the main/master branch of ERPNext directly

For more detailed information, see the [Development Guide](docs/05-development/01-development.md).

## Fixing Duplicate Workspaces

If you accidentally created a new workspace instead of updating an existing one (you see both "ERPNext Settings" and "Villaex Settings" in the sidebar):

### Method 1: Update via UI (Recommended)

1. **Go to:** Workspace → Workspace List
2. **Open "ERPNext Settings"** (the original one)
3. **Change ONLY the Title field** to "Villaex Setting"
4. **IMPORTANT:** Make sure the "Name" field stays as "ERPNext Settings" - do NOT change it
5. **Save**
6. **Delete the duplicate workspace:**
   - Go back to Workspace List
   - Find "Villaex Settings" (the duplicate)
   - Delete it

### Method 2: Update via Bench Console

```bash
cd /workspace/development/frappe-bench
bench --site development.localhost console
```

Then in the Python console:
```python
# Get the existing workspace
workspace = frappe.get_doc("Workspace", "ERPNext Settings")

# Update only title and label (NOT name)
workspace.title = "Villaex Setting"
workspace.label = "Villaex Setting"

# Save
workspace.save()

# Delete the duplicate if it exists
if frappe.db.exists("Workspace", "Villaex Settings"):
    frappe.delete_doc("Workspace", "Villaex Settings", force=1)

# Commit the changes
frappe.db.commit()
```

Then clear cache:
```bash
image.png
```

### Important Notes

- **Never change the `name` field** - it's the document identifier used in URLs
- **Only change `title` and `label`** for display purposes
- If you change `name`, Frappe will create a new document instead of updating the existing one

## Populating Data in ERPNext

There are several ways to populate data in your ERPNext installation:

### Method 1: Use ERPNext Demo Data (Easiest)

ERPNext includes built-in demo data that you can install:

```bash
# Inside your container or dev environment
cd /workspace/development/frappe-bench
bench --site development.localhost execute erpnext.demo.demo_data.make_demo
```

This will create:
- Sample companies, customers, suppliers
- Items/products
- Sales orders, purchase orders
- Invoices and other transactions
- Chart of accounts

### Method 2: Use Data Import Tool (Via UI)

1. **Go to:** Data Import → Data Import
2. **Click "New"** to create a new import
3. **Select the DocType** you want to import (e.g., Customer, Item, etc.)
4. **Download the template** CSV file
5. **Fill in your data** in the CSV
6. **Upload the CSV** and click "Import"

### Method 3: Import from JSON Files

If you have JSON files with data:

```bash
bench --site development.localhost import-doc /path/to/your/data.json
```

### Method 4: Create Data Programmatically (Using Bench Console)

```bash
bench --site development.localhost console
```

Then in the Python console:

```python
# Create a Customer
customer = frappe.get_doc({
    "doctype": "Customer",
    "customer_name": "Test Customer",
    "customer_type": "Company",
    "territory": "All Territories"
})
customer.insert()
frappe.db.commit()
print(f"Created customer: {customer.name}")

# Create an Item
item = frappe.get_doc({
    "doctype": "Item",
    "item_code": "TEST-ITEM-001",
    "item_name": "Test Item",
    "item_group": "Products",
    "stock_uom": "Nos"
})
item.insert()
frappe.db.commit()
print(f"Created item: {item.name}")

# Create a Sales Order
sales_order = frappe.get_doc({
    "doctype": "Sales Order",
    "customer": customer.name,
    "delivery_date": "2024-12-31",
    "items": [{
        "item_code": item.name,
        "qty": 10,
        "rate": 100
    }]
})
sales_order.insert()
frappe.db.commit()
print(f"Created sales order: {sales_order.name}")
```

### Method 5: Use Fixtures (For Development)

Create a Python file with your data:

```python
# my_fixtures.py
import frappe

def execute():
    # Your data creation code here
    customer = frappe.get_doc({
        "doctype": "Customer",
        "customer_name": "My Customer",
        # ... other fields
    })
    customer.insert()
    frappe.db.commit()
```

Then run:
```bash
bench --site development.localhost execute my_fixtures.execute
```

### Quick Demo Data Script

Here's a complete script to populate common data:

```python
# Run in bench console: bench --site development.localhost console

import frappe
from frappe.utils import today, add_days

# Create Customer
customer = frappe.get_doc({
    "doctype": "Customer",
    "customer_name": "Demo Customer",
    "customer_type": "Company",
    "territory": "All Territories"
})
customer.insert()

# Create Supplier
supplier = frappe.get_doc({
    "doctype": "Supplier",
    "supplier_name": "Demo Supplier",
    "supplier_type": "Company"
})
supplier.insert()

# Create Item
item = frappe.get_doc({
    "doctype": "Item",
    "item_code": "DEMO-ITEM-001",
    "item_name": "Demo Product",
    "item_group": "Products",
    "stock_uom": "Nos",
    "is_stock_item": 1
})
item.insert()

frappe.db.commit()
print("✓ Created demo customer, supplier, and item")
```

### Tips

- **Start with demo data** to understand ERPNext structure
- **Use Data Import** for bulk imports from CSV/Excel
- **Use console** for programmatic data creation during development
- **Clear cache** after importing: `bench --site development.localhost clear-cache`

# Documentation

### [Getting Started Guide](docs/getting-started.md)

### [Frequently Asked Questions](https://github.com/frappe/frappe_docker/wiki/Frequently-Asked-Questions)

### [Getting Started](#getting-started)

- [Quick Start (Linux/Mac)](docs/01-getting-started/01-quick-start-linux-mac.md)
- [Single Compose Setup](docs/01-getting-started/02-single-compose-setup.md)

### [Setup](#setup)

- [Container Setup Overview](docs/02-setup/01-overview.md)
- [Build Setup](docs/02-setup/02-build-setup.md)
- [Start Setup](docs/02-setup/03-start-setup.md)
- [Environment Variables](docs/02-setup/04-env-variables.md)
- [Compose Overrides](docs/02-setup/05-overrides.md)
- [Setup Examples](docs/02-setup/06-setup-examples.md)
- [Single Server Example](docs/02-setup/07-single-server-example.md)

### [Production](#production)

- [TLS/SSL Setup](docs/03-production/01-tls-ssl-setup.md)
- [Backup Strategy](docs/03-production/02-backup-strategy.md)
- [Multi-Tenancy](docs/03-production/03-multi-tenancy.md)

### [Operations](#operations)

- [Site Operations](docs/04-operations/01-site-operations.md)

### [Development](#development)

- [Development Guide](docs/05-development/01-development.md)
- [Debugging](docs/05-development/02-debugging.md)
- [Local Services Connection](docs/05-development/03-local-services-connection.md)

### [Migration](#migration)

- [Migrate from Multi-Image Setup](docs/06-migration/01-migrate-from-multi-image-setup.md)

### [Troubleshooting](#troubleshooting)

- [Troubleshoot Guide](docs/07-troubleshooting/01-troubleshoot.md)
- [Windows Nginx Entrypoint Error](docs/07-troubleshooting/02-windows-nginx-entrypoint-error.md)

### [Reference](#reference)

- [Build Version 10 Images](docs/08-reference/01-build-version-10-images.md)

# Contributing

If you want to contribute to this repo refer to [CONTRIBUTING.md](CONTRIBUTING.md)

This repository is only for container related stuff. You also might want to contribute to:

- [Frappe framework](https://github.com/frappe/frappe#contributing),
- [ERPNext](https://github.com/frappe/erpnext#contributing),
- [Frappe Bench](https://github.com/frappe/bench).
