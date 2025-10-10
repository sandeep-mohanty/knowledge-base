# Tutorial: Complete Zanzibar Mapping Generator with Containerization

This comprehensive guide walks through creating a **Python-based Zanzibar Mapping Generator** that uses the **Rich library** for TUI interaction and CSV processing, and then containerizes it using **Docker**.

The tool transforms legacy RBAC definitions into modern **Zanzibar/OpenFGA/SpiceDB authorization mappings** formatted in Markdown for easy documentation and portability.  

It supports:
- **Interactive mode** — enter one mapping at a time.  
- **Bulk mode** — automatically read all mappings from CSV files under the `inputs/` subdirectory.  

Each generated mapping is written into a **Markdown (.md)** file inside the `outputs/` subdirectory.

---

## 1. Folder Structure

Your working directory should look like this:

    .
    ├── Dockerfile
    ├── zanzibar_generator_ui.py
    ├── inputs/
    │   └── legacy_mappings.csv
    └── outputs/

---

## 2. Dependencies

The program requires:

    pip install rich typer pandas

- **Rich**: Provides colorful terminal UI tables and prompts.
- **Typer**: Simplifies CLI and command structure.
- **Pandas**: Efficient CSV reading and parsing.

---

## 3. Python Program Implementation

Create a file named **zanzibar_generator_ui.py** and paste the following:

    import os
    import pandas as pd
    import typer
    from rich.console import Console
    from rich.table import Table
    from rich.prompt import Prompt
    from rich.panel import Panel
    from typing import List

    console = Console()
    app = typer.Typer()

    # --------------- Generate Zanzibar Definition ----------------

    def generate_zanzibar_definition(resource: str, permission: str, roles: List[str]) -> str:
        """
        Build Zanzibar mapping definition block in markdown format.
        """
        resource_name = resource.lower().replace(" ", "_")
        relations = [f"relation {r.lower()}_role: user" for r in roles]
        joined_relations = "\n  ".join(relations)
        permission_logic = " + ".join([f"{r.lower()}_role" for r in roles])

        mapping = (
            f"```zh\n"
            f"definition {resource_name} {{\n"
            f"  {joined_relations}\n\n"
            f"  permission {permission} = {permission_logic}\n"
            f"}}\n"
            f"```"
        )
        return mapping

    def display_result_table(resource: str, permission: str, roles: List[str], mapping: str):
        """
        Display the generated mapping in a formatted Rich table.
        """
        table = Table(title=f"Generated Mapping for {resource}")
        table.add_column("Field", style="cyan", no_wrap=True)
        table.add_column("Details", style="magenta")
        table.add_row("Resource", resource)
        table.add_row("Permission", permission)
        table.add_row("Roles", ", ".join(roles))
        table.add_row("Preview", mapping[:120] + "..." if len(mapping) > 120 else mapping)
        console.print(table)

    def write_to_markdown(mappings: List[str], output_file: str):
        """
        Write all generated mappings into the outputs folder.
        """
        os.makedirs("outputs", exist_ok=True)
        output_path = os.path.join("outputs", output_file)
        with open(output_path, "w", encoding="utf-8") as f:
            f.write("# Zanzibar Mappings Generated from Legacy RBAC\n\n")
            for block in mappings:
                f.write(block + "\n\n")
        console.print(f"[bold green]Mappings saved to outputs/{output_file}[/bold green]")

    # --------------- Interactive Mode ----------------

    def interactive_mode(output_file: str = "zanzibar_mappings.md"):
        console.print(Panel("🧩 [bold cyan]Interactive Zanzibar Mapping Mode[/bold cyan]", expand=False))
        mappings = []
        while True:
            resource = Prompt.ask("Enter Resource (e.g., Users, Organizations)")
            permission = Prompt.ask("Enter Permission (e.g., UserRead, OrgWrite)")
            roles_input = Prompt.ask("Enter Roles (comma-separated, e.g., SA,OA,UA,HELPDESK)")
            roles = [r.strip() for r in roles_input.split(",") if r.strip()]
            mapping = generate_zanzibar_definition(resource, permission, roles)
            display_result_table(resource, permission, roles, mapping)
            mappings.append(mapping)
            cont = Prompt.ask("Add another mapping? (y/n)", default="n")
            if cont.lower().startswith("n"):
                break
        write_to_markdown(mappings, output_file)

    # --------------- Bulk Mode ----------------

    def bulk_mode(output_file: str = "zanzibar_mappings.md"):
        inputs_dir = os.path.join(os.getcwd(), "inputs")
        if not os.path.exists(inputs_dir):
            console.print(f"[red]Error: Missing 'inputs/' subdirectory in {os.getcwd()}[/red]")
            return
        csv_files = [f for f in os.listdir(inputs_dir) if f.lower().endswith(".csv")]
        if not csv_files:
            console.print("[red]No CSV files found in inputs/ directory[/red]")
            return
        console.print(Panel("[bold cyan]Bulk Zanzibar Mapping Mode[/bold cyan]", expand=False))
        console.print("Available CSV files:")
        for i, file in enumerate(csv_files, 1):
            console.print(f"{i}. {file}")
        choice = int(Prompt.ask("Enter the number of the CSV file you want to process"))
        selected_csv = os.path.join(inputs_dir, csv_files[choice - 1])
        df = pd.read_csv(selected_csv)
        if not all(c in df.columns for c in ["Resource", "Permission", "Roles"]):
            console.print("[red]CSV must have headers: Resource, Permission, Roles[/red]")
            return
        mappings = []
        for _, row in df.iterrows():
            resource = str(row["Resource"]).strip()
            permission = str(row["Permission"]).strip()
            roles = [r.strip() for r in str(row["Roles"]).split(",") if r.strip()]
            mapping = generate_zanzibar_definition(resource, permission, roles)
            display_result_table(resource, permission, roles, mapping)
            mappings.append(mapping)
        write_to_markdown(mappings, output_file)

    # --------------- Entry Point ----------------

    @app.command()
    def main():
        console.print(Panel("[bold green]Zanzibar Mapping Generator[/bold green]", expand=False))
        console.print("Choose mode of operation:\n  1. Interactive Mode\n  2. Bulk (CSV) Mode")
        mode = Prompt.ask("Select mode", choices=["1", "2"], default="1")
        if mode == "1":
            interactive_mode()
        else:
            bulk_mode()

    if __name__ == "__main__":
        app()

---

## 4. Dockerfile

Create a **Dockerfile** in the same directory:

    FROM python:3.11-slim
    ENV PYTHONUNBUFFERED=1 PYTHONDONTWRITEBYTECODE=1
    WORKDIR /app
    COPY zanzibar_generator_ui.py .
    RUN pip install --no-cache-dir rich typer pandas
    RUN mkdir -p /app/inputs /app/outputs
    ENTRYPOINT ["python", "zanzibar_generator_ui.py"]

---

## 5. Build the Docker Image

From your terminal, in the same directory as your Dockerfile:

    docker build -t zanzibar-generator .

Confirm it built successfully:

    docker images | grep zanzibar-generator

---

## 6. Run the Program in a Container

### Interactive Mode

Use the program interactively for manual mapping entry:

    docker run -it \
        -v "$(pwd)":/app \
        zanzibar-generator main

You’ll see:

    Choose mode of operation:
      1. Interactive Mode
      2. Bulk (CSV) Mode

Select `1` and start entering your mappings directly.
The generated Markdown file will appear in:

    ./outputs/zanzibar_mappings.md

---

### Bulk Mode

To generate mappings from CSV automatically:

1. Ensure there’s a CSV file under `inputs/` such as:
   
       inputs/legacy_mappings.csv

   The CSV must have these headers:

       Resource,Permission,Roles
       Users,UserRead,SA,OA,UA,HELPDESK
       Organizations,OrgWrite,SA,OA

2. Run the container:

       docker run -it \
           -v "$(pwd)":/app \
           zanzibar-generator main

3. Select option `2` for Bulk (CSV) Mode.  
   The program lists all CSV files inside the `/app/inputs` folder.  
   Select one by number, and a Markdown mapping file will be saved here:

       ./outputs/zanzibar_mappings.md

---

## 7. Verify the Output

Inspect your results with:

    cat outputs/zanzibar_mappings.md

Example content:

    # Zanzibar Mappings Generated from Legacy RBAC

    ```zh
    definition users {
      relation sa_role: user
      relation oa_role: user
      relation ua_role: user
      relation helpdesk_role: user

      permission UserRead = sa_role + oa_role + ua_role + helpdesk_role
    }
    ```

    ```zh
    definition organizations {
      relation sa_role: user
      relation oa_role: user

      permission OrgWrite = sa_role + oa_role
    }
    ```

---

## 8. Optional Docker Compose Configuration

For ease of use, create a `docker-compose.yml`:

    version: "3.9"
    services:
      zanzibar-generator:
        image: zanzibar-generator
        container_name: zanzibar_gen
        stdin_open: true
        tty: true
        working_dir: /app
        volumes:
          - .:/app
        command: ["python", "zanzibar_generator_ui.py", "main"]

Run it with:

    docker-compose run zanzibar-generator

---

## 9. Summary

✅ **Interactive Mode:** Create mappings manually with colored Rich UI  
✅ **Bulk Mode:** Process multiple mappings from CSV under `inputs/`  
✅ **Markdown Output:** Generated files stored in `outputs/` subdirectory  
✅ **Portable Container:** No dependencies to install locally  
✅ **Rich UI:** Clean, intuitive, developer-friendly interface  

This containerized solution provides a reproducible, clean, and portable environment to generate documentation-ready **Zanzibar/OpenFGA/SpiceDB mappings** from legacy RBAC systems.