# Update Permissions YAML

Synchronize the permissions definition from `ai-handbook` into the `users-ms-user-daas-api-server` microservice.

## Source of Truth

- **File:** `docs/fixtures/permissions.yaml` (this repository — ai-handbook)

## Target

- **Repository:** `company/users-ms-user-daas-api-server`
- **Default path inside the microservice:** `apps/user/fixtures/permissions.yaml`

---

## Workflow

### 1. Read Source Permissions

Read `docs/fixtures/permissions.yaml` from the current repository (ai-handbook).
If the file does not exist or is empty, **stop** and inform the user:

> "No se encontró `docs/fixtures/permissions.yaml` en ai-handbook. Asegúrate de que el archivo existe antes de ejecutar este comando."

### 2. Resolve Microservice Location

Ask the user how they want to access the microservice:

**Options:**
- **Ruta local** — The user provides an absolute path to their local clone of `users-ms-user-daas-api-server`.
- **Clonar en /tmp (default)** — Clone the repo into a temporary directory.

If the user does not provide a path, default to cloning:

```bash
TEMP_DIR=$(mktemp -d)
gh repo clone company/users-ms-user-daas-api-server "$TEMP_DIR/users-ms-user-daas-api-server"
MS_DIR="$TEMP_DIR/users-ms-user-daas-api-server"
```

Store the resolved path as `$MS_DIR` for subsequent steps.

### 3. Locate Target Permissions File

Look for the permissions file in the microservice at the default path:

```
$MS_DIR/apps/user/fixtures/permissions.yaml
```

If the file is **not found** at the default path:
1. Search the repo for any `permissions.yaml`: `find $MS_DIR -name "permissions.yaml" -type f`
2. If found elsewhere, confirm with the user: "He encontrado `permissions.yaml` en `<path>`. ¿Es este el archivo correcto?"
3. If not found at all, ask the user to provide the exact path inside the microservice.

### 4. Compare Permissions

Compare the source (ai-handbook) and target (microservice) permissions files.

Run a diff to detect changes:

```bash
diff "$AI_HANDBOOK_DIR/docs/fixtures/permissions.yaml" "$MS_DIR/<target_path>/permissions.yaml"
```

If there are **no differences**, inform the user and stop:

> "Los permisos ya están sincronizados. No hay cambios necesarios."

If there are differences, show the user a **summary** of the changes:
- New permissions added (codenames present in source but not in target)
- Permissions removed (codenames present in target but not in source)
- Permissions modified (same codename, different name/description/scopes)

Ask the user: "¿Deseas aplicar estos cambios?"

### 5. Apply Changes

Once the user confirms:

1. **Create branch** from the microservice's default branch:
   ```bash
   cd "$MS_DIR"
   git checkout -b update-permissions-yaml
   ```

2. **Copy the permissions file** from ai-handbook to the microservice:
   ```bash
   cp "$AI_HANDBOOK_DIR/docs/fixtures/permissions.yaml" "$MS_DIR/<target_path>/permissions.yaml"
   ```

3. **Commit the change:**
   ```bash
   git add <target_path>/permissions.yaml
   git commit -m "$(cat <<'EOF'
   🔧 chore(permissions): sync permissions.yaml from ai-handbook

   Updated permissions definitions to match the source of truth in ai-handbook.

   Refs: update-permissions-yaml
   EOF
   )"
   ```

4. **Push and create PR:**
   ```bash
   git push -u origin update-permissions-yaml
   gh pr create \
     --repo company/users-ms-user-daas-api-server \
     --title "🔧 Sync permissions.yaml from ai-handbook" \
     --body "$(cat <<'EOF'
   ## Summary
   - Synchronized `permissions.yaml` with the source of truth in `ai-handbook`
   - Changes detected and applied automatically via `update-permissions` command

   ## Changes
   <INSERT_DIFF_SUMMARY_HERE>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

### 6. Report Result

Present the result to the user:

```
## Resultado

- ✅ Permisos sincronizados correctamente
- 🔀 Rama: `update-permissions-yaml`
- 🔗 PR: <PR_URL>

### Resumen de cambios
- Permisos añadidos: <count>
- Permisos eliminados: <count>
- Permisos modificados: <count>
```

---

## Important Notes

- The **source of truth** is always `docs/fixtures/permissions.yaml` in ai-handbook. Never modify this file — only push it to the microservice.
- If the microservice was cloned to `/tmp`, remind the user that the temporary clone will be cleaned up on system reboot.
- Always show the diff and get user confirmation before applying changes.
- Follow conventional commit format for the commit message.
- If `gh` CLI is not authenticated, inform the user and stop.
$ARGUMENTS
