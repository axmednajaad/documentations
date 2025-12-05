
# Frappe/ERPNext Site Restoration Guide

This document outlines the step-by-step process for restoring a full site backup (database and files) from one site to a new site on the same Frappe Bench.

## 📝 Prerequisites

1.  Access to the server via SSH.
2.  All commands are executed from the **`~/frappe-bench`** directory.
3.  The source site backup was created using `bench backup --with-files`.

## ⚙️ Define Variables

Before proceeding, set the variables below. Replace the placeholder values with your specific site names and the exact backup timestamp.

The `SOURCE_SITE_UNDERSCORE` variable is crucial as `bench` replaces dots (`.`) with underscores (`_`) in the backup filenames.

```bash
# --- 1. Set Site Names and Timestamp ---

# The name of the new site you are restoring to
TARGET_SITE="axmed.com"

# The name of the original site where the backup was taken
SOURCE_SITE="kulane.kahatech.com"

# The underscore format used in the backup file names (dots replaced with underscores)
SOURCE_SITE_UNDERSCORE="kulane_kahatech_com"

# The EXACT timestamp from your backup output (e.g., 20251205_095322)
TIMESTAMP="20251205_095322"


# --- 2. Define Backup Paths (Do not modify these) ---

BACKUP_DIR="./sites/$SOURCE_SITE/private/backups"

DB_BACKUP_PATH="$BACKUP_DIR/$TIMESTAMP-$SOURCE_SITE_UNDERSCORE-database.sql.gz"
PUBLIC_FILES_ARCHIVE="$BACKUP_DIR/$TIMESTAMP-$SOURCE_SITE_UNDERSCORE-files.tar"
PRIVATE_FILES_ARCHIVE="$BACKUP_DIR/$TIMESTAMP-$SOURCE_SITE_UNDERSCORE-private-files.tar"

# --- 3. Verify Paths (Optional Check) ---
echo "--- Verification ---"
echo "Target Site: $TARGET_SITE"
echo "Database Path: $DB_BACKUP_PATH"
```

## 1\. 🏗️ Create and Prepare the New Site

Create the new site and install the required Frappe applications (e.g., ERPNext).

```bash
# Create the new site
bench new-site $TARGET_SITE --admin-password 'your-strong-password'

# Install the necessary Apps (e.g., erpnext). Repeat for all apps installed on the source site.
# NOTE: This step MUST be done before database restoration.
bench --site $TARGET_SITE install-app erpnext
```

## 2\. 💾 Restore the Database

Use the `bench restore` command with the correctly named database file path.

```bash
echo "Starting Database Restore..."
bench --site $TARGET_SITE restore $DB_BACKUP_PATH
```

## 3\. 📁 Restore Public and Private Files

The file archives must be extracted from the Bench root directory (`~/frappe-bench`) into the site's respective `public` and `private` folders. The `../../../` prefix is used in the `tar` command to correctly navigate from the sub-directories back to the Bench root to find the archive file.

### A. Restore Public Files

```bash
echo "Restoring Public Files..."

# 1. Navigate to the target site's public folder
cd sites/$TARGET_SITE/public/

# 2. Extract public files. --strip-components 4 removes leading directories.
tar -xvf ../../../$PUBLIC_FILES_ARCHIVE --strip-components 4

# 3. Go back to the bench root
cd ../../../
```

### B. Restore Private Files

```bash
echo "Restoring Private Files..."

# 1. Navigate to the target site's private folder
cd sites/$TARGET_SITE/private/

# 2. Extract private files.
tar -xvf ../../../$PRIVATE_FILES_ARCHIVE --strip-components 4

# 3. Go back to the bench root
cd ../../../
```

## 4\. ✅ Finalize and Restart

Run the final migration steps to apply any required database patches, clear the cache, and restart the services.

```bash
echo "Running Final Migrations and Cleanup..."

# Run database migrations and patches
bench --site $TARGET_SITE migrate

# Clear the cache
bench --site $TARGET_SITE clear-cache

# Restart all bench services to apply changes
bench restart
```

-----

## 🚀 Post-Restoration Checklist

1.  **Access:** Log in to the new site using the credentials from the original site.
2.  **Verify Data:** Confirm that master data (e.g., Customers, Items, Accounts) is present.
3.  **Verify Files:** Check the File Manager and check any documents (e.g., Sales Order, Purchase Invoice) to ensure attached files/images are loading correctly.
