# Python Rich Library Tutorial

This tutorial covers the Rich library, which provides beautiful terminal formatting, progress bars, tables, and more for creating stunning command-line interfaces in Python.

## Table of Contents
1. Introduction to Rich
2. Basic Text Formatting
3. The Console Object
4. Tables and Data Display
5. Progress Bars and Status
6. Panels and Layouts
7. Markdown Rendering
8. Syntax Highlighting
9. Tracebacks and Exceptions
10. Rich Logging
11. Inspecting Objects
12. Advanced Usage and Customization

## 1. Introduction to Rich

Rich is a Python library for rich text and beautiful formatting in the terminal. It makes it easy to add color and style to terminal output, as well as provide components like tables, progress bars, and syntax highlighting.

### Installation

```bash
pip install rich
```

### Basic Usage

```python
from rich import print

# Rich print replaces the built-in print with enhanced functionality
print("Hello, [bold red]Rich[/bold red] World!")
print("You can use [blue]colors[/blue], [bold]bold[/bold], [italic]italic[/italic], and more.")
print("[bold yellow on blue]Highlighted text[/bold yellow on blue]")
```

## 2. Basic Text Formatting

Rich uses a simple markup syntax for text styling.

### Text Styles and Colors

```python
from rich import print

# Basic styles
print("[bold]Bold text[/bold]")
print("[italic]Italic text[/italic]")
print("[underline]Underlined text[/underline]")
print("[strike]Strikethrough text[/strike]")

# Colors
print("[red]Red text[/red]")
print("[green]Green text[/green]")
print("[blue]Blue text[/blue]")
print("[yellow]Yellow text[/yellow]")
print("[magenta]Magenta text[/magenta]")
print("[cyan]Cyan text[/cyan]")

# Combining styles
print("[bold red]Bold red text[/bold red]")
print("[italic blue]Italic blue text[/italic blue]")

# Background colors
print("[black on white]Black on white[/black on white]")
print("[white on blue]White on blue[/white on blue]")

# RGB colors
print("[rgb(255,0,0)]Custom red[/rgb(255,0,0)]")
print("[#FF00FF]Hex color (magenta)[/#FF00FF]")
```

### Using the Text Class

```python
from rich.text import Text
from rich.console import Console

console = Console()

# Create a Text object
text = Text("Hello, Rich World!")
text.stylize("bold red", 0, 5)          # Style "Hello"
text.stylize("italic blue", 7, 11)      # Style "Rich"
text.stylize("underline green", 12, 17) # Style "World"

# Display the styled text
console.print(text)

# Text objects can be concatenated
text1 = Text("Bold", style="bold")
text2 = Text(" and ", style="default")
text3 = Text("Italic", style="italic")
combined_text = text1 + text2 + text3

console.print(combined_text)
```

## 3. The Console Object

The Console class is central to Rich's functionality, providing methods for printing formatted text to the terminal.

### Basic Console Usage

```python
from rich.console import Console

# Create a console object
console = Console()

# Basic printing
console.print("Hello, World!")
console.print("This is [bold]bold[/bold] text")

# Print with styles
console.print("Error message", style="bold red")
console.print("Success message", style="green")

# Printing with highlighting
console.print("Important information", style="on yellow")

# Print with alignment
console.print("Centered text", justify="center")
console.print("Right-aligned text", justify="right")

# Print ruler
console.rule("Section Title")

# Print with emoji
console.print(":thumbs_up: Great job!", emoji=True)
```

### Console Options

```python
from rich.console import Console
import sys

# Custom console options
console = Console(
    width=100,               # Set width in characters
    color_system="standard", # "standard", "256", "truecolor", or "auto"
    highlight=True,          # Enable syntax highlighting
    record=True,             # Record output for later export
    file=sys.stderr          # Redirect output to stderr instead of stdout
)

# Print something
console.print("Custom console configuration")

# Export recorded output to HTML
console.save_html("output.html")

# Export recorded output to SVG
console.save_svg("output.svg")
```

### Input and Prompts

```python
from rich.console import Console
from rich.prompt import Prompt, Confirm, IntPrompt

console = Console()

# Basic prompt
name = Prompt.ask("What is your name?")
console.print(f"Hello, {name}!")

# Prompt with default value
language = Prompt.ask("What is your favorite language?", default="Python")
console.print(f"You chose: {language}")

# Prompt with choices
color = Prompt.ask(
    "Choose a color",
    choices=["red", "green", "blue"],
    default="blue"
)
console.print(f"Selected color: [bold {color}]{color}[/bold {color}]")

# Confirmation prompt
if Confirm.ask("Continue?"):
    console.print("Continuing...")
else:
    console.print("Aborted.")

# Integer prompt
age = IntPrompt.ask("How old are you?", default=25)
console.print(f"You are {age} years old")
```

## 4. Tables and Data Display

Rich provides beautiful tables for displaying structured data.

### Basic Table

```python
from rich.console import Console
from rich.table import Table

console = Console()

# Create a table
table = Table(title="Employee Information")

# Add columns with styles
table.add_column("ID", style="cyan", justify="right")
table.add_column("Name", style="magenta")
table.add_column("Department", style="green")
table.add_column("Salary", style="yellow", justify="right")

# Add rows
table.add_row("1", "John Doe", "Engineering", "$75,000")
table.add_row("2", "Jane Smith", "Marketing", "$82,000")
table.add_row("3", "Alice Johnson", "HR", "$67,000")
table.add_row("4", "Bob Brown", "Engineering", "$78,000")

# Print the table
console.print(table)
```

### Table Customization

```python
from rich.console import Console
from rich.table import Table
from rich.style import Style

console = Console()

# Create a customized table
table = Table(
    title="Project Status",
    caption="Updated: August 2025",
    caption_justify="right",
    show_header=True,
    header_style="bold white on blue",
    border_style="bright_black",
    show_lines=True,
    highlight=True,
    box=None,           # None, "ASCII", "SQUARE", "MINIMAL", "SIMPLE", etc.
    safe_box=True,      # Fall back to ASCII if Unicode isn't supported
    expand=False,       # Don't expand to fill terminal width
    width=80            # Fixed width in characters
)

# Add columns
table.add_column("Project", style="cyan")
table.add_column("Status", style="magenta")
table.add_column("Completion", justify="right")

# Define row styles
completed_style = Style(color="green", bold=True)
in_progress_style = Style(color="yellow")
not_started_style = Style(color="red")

# Add rows with different styles
table.add_row("Website Redesign", "Completed", "100%", style=completed_style)
table.add_row("Mobile App", "In Progress", "65%", style=in_progress_style)
table.add_row("API Integration", "In Progress", "42%", style=in_progress_style)
table.add_row("Documentation", "Not Started", "0%", style=not_started_style)

# Print the table
console.print(table)
```

### Using Table with Data

```python
from rich.console import Console
from rich.table import Table
import csv

console = Console()

# Load data from CSV
def load_data(filename):
    data = []
    headers = []
    with open(filename, 'r') as file:
        csv_reader = csv.reader(file)
        headers = next(csv_reader)  # Get the header row
        for row in csv_reader:
            data.append(row)
    return headers, data

# Create and display a table from data
def display_csv_table(filename, title=None):
    try:
        headers, data = load_data(filename)
        
        table = Table(title=title or filename)
        
        # Add columns
        for header in headers:
            table.add_column(header)
        
        # Add rows
        for row in data:
            table.add_row(*row)
        
        console.print(table)
        return True
    except Exception as e:
        console.print(f"Error loading CSV: {e}", style="bold red")
        return False

# Example usage (you'll need a CSV file or can use this code with a real file)
# display_csv_table("data.csv", "Sales Data")

# Alternatively, create a table with Python data
def display_python_data(data, columns, title=None):
    table = Table(title=title)
    
    # Add columns
    for column in columns:
        table.add_column(column)
    
    # Add rows
    for row in data:
        # Convert all values to strings
        string_row = [str(item) for item in row]
        table.add_row(*string_row)
    
    console.print(table)

# Example with Python data
sales_data = [
    [1, "North", 10245, 5.2],
    [2, "South", 8917, -2.7],
    [3, "East", 12603, 7.1],
    [4, "West", 9002, 1.5]
]

columns = ["Region ID", "Region Name", "Sales", "Growth (%)"]
display_python_data(sales_data, columns, "Regional Sales Data")
```

## 5. Progress Bars and Status

Rich can display beautiful progress bars and status indicators for long-running tasks.

### Basic Progress Bar

```python
from rich.progress import Progress, TextColumn, BarColumn, TimeElapsedColumn, TimeRemainingColumn
import time

# Simple progress bar
with Progress() as progress:
    task = progress.add_task("[green]Processing...", total=100)
    
    for i in range(100):
        time.sleep(0.05)  # Simulate work
        progress.update(task, advance=1)

# Custom progress bar columns
with Progress(
    TextColumn("[bold blue]{task.description}"),
    BarColumn(bar_width=40),
    TextColumn("[progress.percentage]{task.percentage:>3.0f}%"),
    TimeElapsedColumn(),
    TimeRemainingColumn()
) as progress:
    task = progress.add_task("[red]Downloading...", total=1000)
    
    for i in range(1000):
        time.sleep(0.01)  # Simulate work
        progress.update(task, advance=1)
```

### Multiple Progress Bars

```python
from rich.progress import Progress, TextColumn, BarColumn, SpinnerColumn
import time
import random

# Multiple progress bars
with Progress(
    SpinnerColumn(),
    TextColumn("[bold blue]{task.description}"),
    BarColumn(),
    TextColumn("[progress.percentage]{task.percentage:>3.0f}%"),
    TextColumn("[bold green]{task.completed}/{task.total}")
) as progress:
    task1 = progress.add_task("[red]Downloading...", total=1000)
    task2 = progress.add_task("[green]Processing...", total=725)
    task3 = progress.add_task("[yellow]Installing...", total=400)
    
    while not progress.finished:
        progress.update(task1, advance=random.uniform(0, 5))
        progress.update(task2, advance=random.uniform(0, 3))
        progress.update(task3, advance=random.uniform(0, 1))
        time.sleep(0.02)
```

### Progress with Iteration

```python
from rich.progress import track
import time

# Iterate over a sequence with a progress bar
for i in track(range(50), description="Processing..."):
    time.sleep(0.1)  # Simulate work

# Process a list of items with a progress bar
items = ["item1", "item2", "item3", "item4", "item5", "item6", "item7", "item8"]

def process_item(item):
    """Process a single item."""
    time.sleep(0.5)  # Simulate processing
    return f"Processed {item}"

results = []
for item in track(items, description="Processing items"):
    result = process_item(item)
    results.append(result)

# Print results
from rich.console import Console
console = Console()
console.print(results)
```

### Status and Spinners

```python
from rich.console import Console
from rich.status import Status
import time

console = Console()

# Simple status
with Status("Working...") as status:
    time.sleep(2)
    status.update(status="Still working...")
    time.sleep(2)
    status.update(status="Almost done...")
    time.sleep(2)

# Custom spinner
with Status("Loading...", spinner="dots") as status:
    time.sleep(3)

# Different spinner styles
spinners = ["dots", "line", "star", "arrow", "bouncingBar", "bouncingBall"]

for spinner in spinners:
    with Status(f"Using '{spinner}' spinner", spinner=spinner) as status:
        time.sleep(2)
```

## 6. Panels and Layouts

Rich can create panels to group content and arrange multiple panels in layouts.

### Basic Panels

```python
from rich.console import Console
from rich.panel import Panel
from rich.text import Text

console = Console()

# Simple panel
console.print(Panel("This is a panel"))

# Styled panel
console.print(
    Panel(
        "This panel has a title and custom style",
        title="Title",
        subtitle="Subtitle",
        border_style="green",
        padding=(1, 2)
    )
)

# Panel with multiple lines
panel_content = (
    "Panels are useful for grouping content.\n\n"
    "You can add multiple paragraphs and style the content."
)
console.print(Panel(panel_content, title="About Panels"))

# Panel with styled content
text = Text.from_markup(
    "[bold]Rich[/bold] makes it easy to create [yellow]beautiful[/yellow] terminal output."
)
console.print(Panel(text, title="Styled Content"))
```

### Using Layouts

```python
from rich.console import Console
from rich.panel import Panel
from rich.layout import Layout
from rich.text import Text

console = Console()

# Create a layout
layout = Layout()

# Split layout into sections
layout.split(
    Layout(name="header", size=3),
    Layout(name="main", ratio=1),
    Layout(name="footer", size=3)
)

# Split the main section into columns
layout["main"].split_row(
    Layout(name="left", ratio=1),
    Layout(name="right", ratio=2)
)

# Add content to sections
layout["header"].update(Panel("Header", border_style="blue"))
layout["footer"].update(Panel("Footer", border_style="blue"))
layout["left"].update(Panel("Left Sidebar", border_style="green"))
layout["right"].update(Panel("Main Content Area", border_style="red"))

# Print the layout
console.print(layout)
```

### Complex Layouts

```python
from rich.console import Console
from rich.panel import Panel
from rich.layout import Layout
from rich.text import Text
from rich.table import Table
from datetime import datetime

console = Console()

# Create a layout for a dashboard
def make_dashboard():
    # Create layout
    layout = Layout(name="root")
    
    # Split the root layout into header, body, and footer
    layout.split(
        Layout(name="header", size=3),
        Layout(name="body", ratio=1),
        Layout(name="footer", size=3)
    )
    
    # Split the body into sidebar and content
    layout["body"].split_row(
        Layout(name="sidebar", ratio=1),
        Layout(name="content", ratio=3)
    )
    
    # Split content into upper and lower
    layout["content"].split(
        Layout(name="upper_content", ratio=2),
        Layout(name="lower_content", ratio=1)
    )
    
    # Add header content
    current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    layout["header"].update(
        Panel(
            f"System Dashboard - {current_time}",
            style="white on blue",
            border_style="blue"
        )
    )
    
    # Add sidebar content - navigation
    nav_items = [
        "[x] Dashboard",
        "[ ] Users",
        "[ ] Settings",
        "[ ] Reports"
    ]
    nav_text = Text("\n".join(nav_items))
    layout["sidebar"].update(
        Panel(
            nav_text,
            title="Navigation",
            border_style="green"
        )
    )
    
    # Add upper content - system stats
    stats_table = Table(show_header=True, header_style="bold")
    stats_table.add_column("Metric")
    stats_table.add_column("Value")
    
    stats_table.add_row("CPU Usage", "45%")
    stats_table.add_row("Memory Usage", "2.3 GB / 8 GB")
    stats_table.add_row("Disk Space", "120 GB / 500 GB")
    stats_table.add_row("Network", "28 MB/s")
    
    layout["upper_content"].update(
        Panel(
            stats_table,
            title="System Statistics",
            border_style="red"
        )
    )
    
    # Add lower content - recent logs
    logs = [
        "[green]INFO[/green] System started successfully",
        "[yellow]WARNING[/yellow] High memory usage detected",
        "[green]INFO[/green] Backup completed",
        "[red]ERROR[/red] Failed to connect to database"
    ]
    log_text = Text("\n".join(logs))
    layout["lower_content"].update(
        Panel(
            log_text,
            title="Recent Logs",
            border_style="yellow"
        )
    )
    
    # Add footer
    layout["footer"].update(
        Panel(
            "Press Q to quit, H for help",
            style="white on blue",
            border_style="blue"
        )
    )
    
    return layout

# Print the dashboard
console.print(make_dashboard())
```

## 7. Markdown Rendering

Rich can render Markdown text directly in the terminal.

### Basic Markdown

```python
from rich.console import Console
from rich.markdown import Markdown

console = Console()

# Simple markdown
markdown_text = """
# Rich Markdown Example

This is a demonstration of Rich's markdown rendering capabilities.

## Features

* **Bold text** and *italic text*
* ~~Strikethrough~~
* [Links](https://github.com/Textualize/rich)
* Lists and code blocks

## Code Example

```python
def hello_rich():
    print("Hello, Rich!")
```

> Rich supports blockquotes too.

---

1. First ordered item
2. Second ordered item
3. Third ordered item
"""

# Render markdown
md = Markdown(markdown_text)
console.print(md)
```

### Customizing Markdown Rendering

```python
from rich.console import Console
from rich.markdown import Markdown

console = Console()

# Markdown with custom styles
markdown_text = """
# Custom Styled Markdown

This markdown will use custom styling.

## Code blocks with syntax highlighting

```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n-1)
```

## Lists with custom bullets

* First item
* Second item
* Third item
"""

# Define custom styles for markdown elements
md = Markdown(
    markdown_text,
    code_theme="monokai",    # Syntax highlighting theme
    inline_code_theme=None,  # Theme for inline code or None for default
    hyperlinks=True          # Enable clickable links in supported terminals
)

console.print(md)
```

### Loading Markdown from File

```python
from rich.console import Console
from rich.markdown import Markdown
import pathlib

console = Console()

def render_markdown_file(filename):
    """Render a markdown file in the terminal."""
    try:
        path = pathlib.Path(filename)
        markdown_text = path.read_text()
        md = Markdown(markdown_text)
        console.print(md)
        return True
    except Exception as e:
        console.print(f"Error rendering markdown file: {e}", style="bold red")
        return False

# Example usage
# render_markdown_file("README.md")

# Alternatively, render markdown from a string
sample_md = """
# Hello, Rich!

This is some **markdown** that we can *render* directly.

## Code Sample

```python
print("Hello, World!")
```
"""

console.print(Markdown(sample_md))
```

## 8. Syntax Highlighting

Rich provides powerful syntax highlighting for code.

### Basic Syntax Highlighting

```python
from rich.console import Console
from rich.syntax import Syntax

console = Console()

# Python code with syntax highlighting
python_code = '''
def factorial(n):
    """Calculate factorial of n."""
    if n <= 1:
        return 1
    return n * factorial(n-1)

# Calculate factorial of 5
result = factorial(5)
print(f"5! = {result}")
'''

syntax = Syntax(python_code, "python", theme="monokai", line_numbers=True)
console.print(syntax)

# Highlight other languages
javascript_code = '''
function greet(name) {
    return `Hello, ${name}!`;
}

const message = greet("World");
console.log(message);
'''

console.print(Syntax(javascript_code, "javascript", theme="monokai", line_numbers=True))
```

### Customizing Syntax Highlighting

```python
from rich.console import Console
from rich.syntax import Syntax

console = Console()

# Python code
code = '''
def hello(name):
    """Say hello to someone."""
    return f"Hello, {name}!"

# Main program
if __name__ == "__main__":
    message = hello("Rich")
    print(message)
'''

# Basic syntax highlighting
syntax1 = Syntax(
    code,
    "python",
    theme="monokai",           # Choose a color theme
    line_numbers=True,         # Show line numbers
    word_wrap=True,            # Wrap long lines
    indent_guides=True,        # Show indent guides
    highlight_lines=[5, 6]     # Highlight specific lines
)

console.print(syntax1)
console.print()

# More customization options
syntax2 = Syntax(
    code,
    "python",
    theme="github-dark",       # Different theme
    line_numbers=True,
    code_width=60,             # Maximum width
    tab_size=4,                # Tab size in spaces
    background_color="black",  # Override background
    line_range=(3, 8)          # Only show these lines
)

console.print(syntax2)
```

### Highlighting Files

```python
from rich.console import Console
from rich.syntax import Syntax
import pathlib

console = Console()

def highlight_file(filename, language=None):
    """Highlight the syntax of a file."""
    try:
        path = pathlib.Path(filename)
        code = path.read_text()
        
        # Auto-detect language from file extension if not specified
        if language is None:
            language = path.suffix.lstrip('.')
            if language == 'py':
                language = 'python'
            elif language == 'js':
                language = 'javascript'
            elif language == 'md':
                language = 'markdown'
            # Add more mappings as needed
        
        syntax = Syntax(
            code,
            language,
            theme="monokai",
            line_numbers=True,
            word_wrap=True,
            indent_guides=True
        )
        
        console.print(syntax)
        return True
    except Exception as e:
        console.print(f"Error highlighting file: {e}", style="bold red")
        return False

# Example usage
# highlight_file("example.py")
# highlight_file("example.js", "javascript")

# Alternatively, print an example
example_html = '''
<!DOCTYPE html>
<html>
<head>
    <title>Rich Syntax Highlighting</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        h1 {
            color: navy;
        }
    </style>
</head>
<body>
    <h1>Hello, Rich!</h1>
    <p>This is an example of HTML syntax highlighting.</p>
</body>
</html>
'''

console.print(Syntax(example_html, "html", theme="monokai", line_numbers=True))
```

## 9. Tracebacks and Exceptions

Rich can display beautiful tracebacks for Python exceptions.

### Basic Traceback Handling

```python
from rich.console import Console
from rich.traceback import install

# Install rich as the default traceback handler
install(show_locals=True)

console = Console()

# Function with an error
def calculate_average(numbers):
    total = sum(numbers)
    return total / len(numbers)

# This will cause an error
try:
    result = calculate_average([])
except Exception:
    console.print_exception()
```

### Customizing Traceback Display

```python
from rich.console import Console
from rich.traceback import Traceback
import traceback
import sys

console = Console()

# Function that will raise an exception
def level3():
    x = 1 / 0

def level2():
    level3()

def level1():
    level2()

# Capture and display a traceback
try:
    level1()
except Exception:
    # Get exception info
    exc_type, exc_value, exc_traceback = sys.exc_info()
    
    # Create a custom traceback
    tb = Traceback.from_exception(
        exc_type,
        exc_value,
        exc_traceback,
        width=100,           # Width of the traceback
        extra_lines=3,       # Context lines before and after error
        theme="monokai",     # Syntax theme
        word_wrap=True,      # Enable word wrapping
        show_locals=True,    # Show local variables
        max_frames=10        # Maximum number of frames to show
    )
    
    # Print the traceback
    console.print(tb)
```

### Capturing and Saving Tracebacks

```python
from rich.console import Console
from rich.traceback import Traceback
import traceback
import sys

console = Console(record=True)

# Function with a bug
def parse_data(data):
    result = {}
    for item in data:
        key, value = item.split(':')
        result[key.strip()] = value.strip()
    return result

# Try to run the function with bad data
try:
    data = ["name: Alice", "age: 30", "occupation"]  # Missing colon
    parse_data(data)
except Exception:
    # Get exception info
    exc_type, exc_value, exc_traceback = sys.exc_info()
    
    # Create and print traceback
    tb = Traceback.from_exception(
        exc_type,
        exc_value,
        exc_traceback,
        show_locals=True
    )
    console.print(tb)
    
    # Save the traceback to a file
    console.save_html("error_report.html")
    
    # You could also log to a file
    with open("error_log.txt", "w") as file:
        file.write(str(tb))
```

## 10. Rich Logging

Rich provides enhanced logging that integrates with Python's logging module.

### Basic Logging

```python
import logging
from rich.logging import RichHandler
from rich.console import Console
import time

# Configure rich logging
logging.basicConfig(
    level=logging.INFO,
    format="%(message)s",
    datefmt="[%X]",
    handlers=[RichHandler(rich_tracebacks=True)]
)

# Get a logger
log = logging.getLogger("rich")

# Log messages
log.info("Server starting...")
log.debug("Debug information")
log.warning("This is a warning")
log.error("An error occurred")

try:
    # Generate an exception
    x = 1 / 0
except Exception:
    log.exception("An exception occurred")
```

### Customized Logging

```python
import logging
from rich.logging import RichHandler
from rich.console import Console
import time

# Create console with options
console = Console(width=120, color_system="truecolor")

# Configure custom rich handler
rich_handler = RichHandler(
    console=console,
    rich_tracebacks=True,
    tracebacks_show_locals=True,
    show_time=True,
    show_path=True,
    enable_link_path=True
)

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,
    format="%(name)s - %(message)s",
    datefmt="[%X]",
    handlers=[rich_handler]
)

# Create a logger
logger = logging.getLogger("app")

# Log messages with various levels
logger.debug("Starting application")
logger.info("Application is running")
logger.warning("Memory usage is high")
logger.error("Failed to connect to database")
logger.critical("System is unstable")

# Log with extra data
extra_data = {"user": "admin", "ip": "192.168.1.1"}
logger.info("User logged in", extra={"extra": extra_data})

# Log an exception
try:
    result = 10 / 0
except Exception:
    logger.exception("An error occurred during calculation")
```

### Logging to File and Console

```python
import logging
from rich.logging import RichHandler
from rich.console import Console
import time

# Create rich handler for console output
rich_handler = RichHandler(
    rich_tracebacks=True,
    show_time=True,
    show_path=True
)

# Create file handler for file output
file_handler = logging.FileHandler("app.log")
file_format = logging.Formatter(
    "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
file_handler.setFormatter(file_format)

# Configure logging
logger = logging.getLogger("dual_output")
logger.setLevel(logging.DEBUG)
logger.addHandler(rich_handler)
logger.addHandler(file_handler)

# Log some messages
logger.debug("This is a debug message")
logger.info("This is an info message")
logger.warning("This is a warning message")
logger.error("This is an error message")

# Log an exception
try:
    # Generate an exception
    x = 1 / 0
except Exception:
    logger.exception("An exception occurred")

print("Check app.log for the file output")
```

## 11. Inspecting Objects

Rich can help inspect Python objects and data structures.

### Basic Object Inspection

```python
from rich.console import Console
from rich import inspect
from rich.pretty import pprint

console = Console()

# Define a complex object
class Person:
    def __init__(self, name, age, address):
        self.name = name
        self.age = age
        self.address = address
        self._private = "hidden"
    
    def greet(self):
        return f"Hello, my name is {self.name}"

# Create an instance
person = Person("Alice", 30, {"street": "123 Main St", "city": "New York", "zip": "10001"})

# Basic inspection
console.print("Basic inspect:")
inspect(person)

# Customized inspection
console.print("\nCustomized inspect:")
inspect(person, methods=True, private=True, dunder=False, help=False, title="Person Object")

# Pretty printing
console.print("\nPretty print:")
data = {
    "name": "Alice",
    "age": 30,
    "skills": ["Python", "JavaScript", "SQL"],
    "projects": [
        {"name": "Website", "status": "completed"},
        {"name": "API", "status": "in progress"}
    ]
}
pprint(data)
```

### Advanced Object Inspection

```python
from rich.console import Console
from rich import inspect
from rich.pretty import Pretty
import datetime
import dataclasses

console = Console()

# Define a dataclass
@dataclasses.dataclass
class Project:
    name: str
    language: str
    lines_of_code: int
    last_modified: datetime.datetime
    contributors: list

# Create a complex object
project = Project(
    name="Rich Tutorial",
    language="Python",
    lines_of_code=1200,
    last_modified=datetime.datetime.now(),
    contributors=["Alice", "Bob", "Charlie"]
)

# Inspect with various options
console.print("Default inspect:")
inspect(project)

console.print("\nDetailed inspect:")
inspect(project, all=True, value=True)

# Pretty print with custom options
pretty = Pretty(
    project,
    indent_guides=True,
    expand_all=True,
    max_depth=2
)
console.print("\nCustom pretty print:")
console.print(pretty)

# Inspect module
console.print("\nInspecting a module:")
import rich.console
inspect(rich.console, methods=True, help=True)
```

### Inspecting Collections and Complex Data

```python
from rich.console import Console
from rich.pretty import pprint
from rich.tree import Tree
import json

console = Console()

# Complex nested data
complex_data = {
    "users": [
        {
            "id": 1,
            "name": "Alice",
            "roles": ["admin", "developer"],
            "metadata": {
                "joined": "2022-01-15",
                "last_login": "2022-08-29"
            }
        },
        {
            "id": 2,
            "name": "Bob",
            "roles": ["user"],
            "metadata": {
                "joined": "2022-03-20",
                "last_login": "2022-08-27"
            }
        }
    ],
    "settings": {
        "theme": "dark",
        "notifications": True,
        "max_items": 100
    },
    "stats": {
        "visitors": 12345,
        "uptime": "99.9%",
        "version": "1.2.3"
    }
}

# Pretty print the data
console.print("Pretty printing complex data:")
pprint(complex_data)

# Create a tree representation
console.print("\nTree representation:")
tree = Tree("📁 Data")

# Add users branch
users_branch = tree.add("👥 Users")
for user in complex_data["users"]:
    user_branch = users_branch.add(f"👤 {user['name']} (ID: {user['id']})")
    user_branch.add(f"🔑 Roles: {', '.join(user['roles'])}")
    metadata_branch = user_branch.add("📋 Metadata")
    for key, value in user["metadata"].items():
        metadata_branch.add(f"{key}: {value}")

# Add settings branch
settings_branch = tree.add("⚙️ Settings")
for key, value in complex_data["settings"].items():
    settings_branch.add(f"{key}: {value}")

# Add stats branch
stats_branch = tree.add("📊 Stats")
for key, value in complex_data["stats"].items():
    stats_branch.add(f"{key}: {value}")

# Print the tree
console.print(tree)

# Convert to JSON and highlight
console.print("\nJSON representation:")
json_str = json.dumps(complex_data, indent=2)
from rich.syntax import Syntax
console.print(Syntax(json_str, "json", theme="monokai"))
```

## 12. Advanced Usage and Customization

### Creating Custom Rich Components

```python
from rich.console import Console, ConsoleOptions, RenderResult
from rich.panel import Panel
from rich.text import Text
from rich.table import Table
from rich import box
from datetime import datetime
import time

console = Console()

# Create a custom renderable component
class StatusCard:
    """A custom renderable component for system status."""
    
    def __init__(self, title, status, metrics=None, updated_at=None):
        self.title = title
        self.status = status
        self.metrics = metrics or {}
        self.updated_at = updated_at or datetime.now()
    
    def __rich_console__(self, console, options):
        # Create a table for the metrics
        metrics_table = Table(
            show_header=False,
            box=None,
            padding=(0, 1)
        )
        metrics_table.add_column("Key")
        metrics_table.add_column("Value")
        
        for key, value in self.metrics.items():
            metrics_table.add_row(key, str(value))
        
        # Create a status text with appropriate color
        status_style = {
            "online": "bold green",
            "degraded": "bold yellow",
            "offline": "bold red"
        }.get(self.status.lower(), "bold")
        
        status_text = Text(f"Status: {self.status}", style=status_style)
        
        # Create the content
        content = Text(f"{self.title}\n\n")
        content.append(status_text)
        content.append("\n\n")
        content.append(metrics_table)
        content.append(f"\n\nLast updated: {self.updated_at.strftime('%Y-%m-%d %H:%M:%S')}")
        
        # Set the border color based on status
        border_style = {
            "online": "green",
            "degraded": "yellow",
            "offline": "red"
        }.get(self.status.lower(), "blue")
        
        # Return a panel
        yield Panel(
            content,
            title=f"📊 {self.title}",
            border_style=border_style,
            padding=(1, 2),
            box=box.ROUNDED
        )

# Use the custom component
api_status = StatusCard(
    "API Server",
    "Online",
    {
        "Uptime": "99.9%",
        "Response Time": "120ms",
        "Requests/sec": "250",
        "Error Rate": "0.2%"
    }
)

database_status = StatusCard(
    "Database",
    "Degraded",
    {
        "Uptime": "98.5%",
        "Query Time": "350ms",
        "Connections": "42/100",
        "Disk Usage": "78%"
    }
)

cache_status = StatusCard(
    "Cache Server",
    "Offline",
    {
        "Uptime": "0%",
        "Memory Usage": "0%",
        "Hit Rate": "0%",
        "Items": "0"
    }
)

# Print the components
console.print(api_status)
console.print()
console.print(database_status)
console.print()
console.print(cache_status)
```

### Theme Customization

```python
from rich.console import Console
from rich.theme import Theme
from rich.text import Text
from rich.panel import Panel

# Define a custom theme
custom_theme = Theme({
    "info": "dim blue",
    "warning": "magenta",
    "danger": "bold red",
    "success": "green",
    "command": "bold cyan",
    "header": "yellow underline",
    "filename": "#FF6B6B italic",
    "filepath": "#4ECDC4",
    "timestamp": "#A2FAA3"
})

# Create a console with the custom theme
console = Console(theme=custom_theme)

# Use the custom theme styles
console.print("[info]This is information[/info]")
console.print("[warning]This is a warning[/warning]")
console.print("[danger]This is danger![/danger]")
console.print("[success]This is a success message[/success]")
console.print("[command]python script.py --option value[/command]")
console.print("[header]Section Title[/header]")
console.print(f"Editing file [filename]main.py[/filename] in [filepath]/home/user/project/[/filepath]")
console.print(f"Last modified: [timestamp]2025-08-30 15:42:13[/timestamp]")

# Combine styles in a panel
content = Text.from_markup(
    "Running [command]python app.py[/command]\n\n"
    "[success]Server started successfully![/success]\n"
    "Listening on [info]http://localhost:8000[/info]\n\n"
    "[warning]Disk space is running low[/warning]"
)

console.print(Panel(content, title="Application Status", border_style="blue"))
```

### Creating a Rich CLI Application

```python
import sys
import time
from rich.console import Console
from rich.panel import Panel
from rich.progress import Progress, SpinnerColumn, TextColumn, BarColumn
from rich.prompt import Prompt, Confirm
from rich.table import Table
from rich.syntax import Syntax
from rich.layout import Layout
from rich.markdown import Markdown

console = Console()

# Helper function to create a header
def create_header(title):
    return Panel(f"[bold blue]{title}[/bold blue]", border_style="blue")

# Helper function to create a footer
def create_footer():
    return Panel("[bold]q[/bold]: Quit | [bold]h[/bold]: Help", border_style="blue")

# Main application class
class RichCLIApp:
    def __init__(self):
        self.console = Console()
        self.running = True
    
    def show_welcome(self):
        """Display welcome screen"""
        self.console.clear()
        self.console.print(create_header("Rich CLI Demo Application"))
        
        welcome_md = """
        # Welcome to the Rich CLI Demo!

        This application demonstrates various Rich features in a command-line interface.

        ## Available Options

        1. **View System Status** - Display system information
        2. **File Viewer** - View and syntax highlight files
        3. **Data Analysis** - Analyze sample data
        4. **Run Task** - Run a simulated task with progress
        5. **Help** - Show help information
        6. **Exit** - Exit the application
        """
        
        self.console.print(Markdown(welcome_md))
        self.console.print(create_footer())
    
    def show_system_status(self):
        """Display system status information"""
        self.console.clear()
        self.console.print(create_header("System Status"))
        
        # Create a table for system info
        table = Table(show_header=True, header_style="bold blue")
        table.add_column("Component", style="dim")
        table.add_column("Status", style="dim")
        table.add_column("Details", style="dim")
        
        # Add rows with example data
        table.add_row("CPU", "[green]Normal[/green]", "Usage: 32%")
        table.add_row("Memory", "[yellow]Warning[/yellow]", "Usage: 78% (5.6 GB / 8 GB)")
        table.add_row("Disk", "[green]Normal[/green]", "Usage: 45% (230 GB / 500 GB)")
        table.add_row("Network", "[green]Normal[/green]", "10 Mbps")
        table.add_row("Temperature", "[green]Normal[/green]", "42°C")
        
        self.console.print(table)
        
        # Wait for input
        self.console.print()
        Prompt.ask("Press [bold]Enter[/bold] to return to menu")
    
    def file_viewer(self):
        """Simple file viewer with syntax highlighting"""
        self.console.clear()
        self.console.print(create_header("File Viewer"))
        
        # Example files
        example_files = {
            "1": ("Python Script", "python", '''
def greet(name):
    """Greet someone by name"""
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("Rich User"))
'''),
            "2": ("HTML Document", "html", '''
<!DOCTYPE html>
<html>
<head>
    <title>Example Page</title>
    <style>
        body { font-family: Arial, sans-serif; }
        h1 { color: navy; }
    </style>
</head>
<body>
    <h1>Hello World</h1>
    <p>This is an example HTML document.</p>
</body>
</html>
'''),
            "3": ("JSON Data", "json", '''
{
    "user": {
        "name": "John Doe",
        "age": 30,
        "active": true,
        "roles": ["admin", "user"],
        "address": {
            "street": "123 Main St",
            "city": "Anytown",
            "zip": "12345"
        }
    }
}
''')
        }
        
        # Display file options
        self.console.print("Available files to view:")
        for key, (name, _, _) in example_files.items():
            self.console.print(f"  {key}. {name}")
        
        # Get file choice
        choice = Prompt.ask("Choose a file", choices=list(example_files.keys()) + ["b"], default="1")
        
        if choice == "b":
            return  # Back to menu
        
        # Display the chosen file
        name, language, content = example_files[choice]
        self.console.clear()
        self.console.print(create_header(f"File: {name}"))
        
        syntax = Syntax(content.strip(), language, theme="monokai", line_numbers=True, indent_guides=True)
        self.console.print(syntax)
        
        # Wait for input
        self.console.print()
        Prompt.ask("Press [bold]Enter[/bold] to return to menu")
    
    def data_analysis(self):
        """Show a data analysis demo"""
        self.console.clear()
        self.console.print(create_header("Data Analysis"))
        
        # Create a table with sample data
        table = Table(title="Sales Data by Region")
        table.add_column("Region", style="cyan")
        table.add_column("Q1", justify="right", style="green")
        table.add_column("Q2", justify="right", style="green")
        table.add_column("Q3", justify="right", style="green")
        table.add_column("Q4", justify="right", style="green")
        table.add_column("Total", justify="right", style="bold green")
        
        # Add data
        table.add_row("North", "$10,200", "$12,450", "$14,800", "$16,000", "$53,450")
        table.add_row("South", "$8,500", "$9,600", "$10,200", "$11,500", "$39,800")
        table.add_row("East", "$12,300", "$13,450", "$15,230", "$16,800", "$57,780")
        table.add_row("West", "$9,800", "$10,200", "$11,500", "$12,100", "$43,600")
        
        self.console.print(table)
        
        # Add some analysis
        self.console.print()
        self.console.print("[bold]Summary Analysis:[/bold]")
        self.console.print("• East region shows the [bold green]highest[/bold green] total sales")
        self.console.print("• All regions show consistent [bold]growth[/bold] through the quarters")
        self.console.print("• Q4 performance is [bold green]strongest[/bold green] across all regions")
        
        # Wait for input
        self.console.print()
        Prompt.ask("Press [bold]Enter[/bold] to return to menu")
    
    def run_task(self):
        """Run a simulated task with progress display"""
        self.console.clear()
        self.console.print(create_header("Task Runner"))
        
        if not Confirm.ask("Start the simulated task?"):
            return
        
        # Run a multi-stage task with progress
        with Progress(
            SpinnerColumn(),
            TextColumn("[bold blue]{task.description}"),
            BarColumn(),
            TextColumn("[progress.percentage]{task.percentage:>3.0f}%"),
            TextColumn("•"),
            TimeElapsedColumn(),
            console=self.console
        ) as progress:
            # Task 1: Initialization
            init_task = progress.add_task("[green]Initializing...", total=100)
            for i in range(100):
                time.sleep(0.02)
                progress.update(init_task, advance=1)
            
            # Task 2: Data processing
            process_task = progress.add_task("[yellow]Processing data...", total=200)
            for i in range(200):
                time.sleep(0.015)
                progress.update(process_task, advance=1)
            
            # Task 3: Finalization
            final_task = progress.add_task("[cyan]Finalizing...", total=50)
            for i in range(50):
                time.sleep(0.04)
                progress.update(final_task, advance=1)
        
        # Show completion
        self.console.print("[bold green]✓[/bold green] Task completed successfully!")
        
        # Wait for input
        self.console.print()
        Prompt.ask("Press [bold]Enter[/bold] to return to menu")
    
    def show_help(self):
        """Display help information"""
        self.console.clear()
        self.console.print(create_header("Help Information"))
        
        help_md = """
        # Rich CLI Demo Help

        This application demonstrates various Rich library features in a command-line interface.

        ## Navigation

        - Use the number keys (1-6) to select menu options
        - Press 'q' at any time to quit the application
        - Press 'Enter' to confirm selections or return to menus

        ## Features Demonstrated

        - **Rich Text Formatting** - Colors, styles, and formatting
        - **Tables** - Tabular data with styling
        - **Panels** - Bordered content with titles
        - **Progress Bars** - Visual progress indicators
        - **Syntax Highlighting** - Code with color highlighting
        - **Markdown** - Rendering markdown in the terminal

        ## About Rich Library

        Rich is a Python library for rich text and beautiful formatting in the terminal.
        Learn more at: [https://github.com/Textualize/rich](https://github.com/Textualize/rich)
        """
        
        self.console.print(Markdown(help_md))
        
        # Wait for input
        self.console.print()
        Prompt.ask("Press [bold]Enter[/bold] to return to menu")
    
    def run(self):
        """Main application loop"""
        try:
            while self.running:
                self.show_welcome()
                
                choice = Prompt.ask(
                    "Choose an option",
                    choices=["1", "2", "3", "4", "5", "6", "q", "h"],
                    default="1"
                )
                
                if choice == "1":
                    self.show_system_status()
                elif choice == "2":
                    self.file_viewer()
                elif choice == "3":
                    self.data_analysis()
                elif choice == "4":
                    self.run_task()
                elif choice == "5" or choice == "h":
                    self.show_help()
                elif choice == "6" or choice == "q":
                    if Confirm.ask("Are you sure you want to exit?"):
                        self.running = False
                        self.console.print("[bold green]Thank you for using the Rich CLI Demo![/bold green]")
                        break
        
        except KeyboardInterrupt:
            self.console.print("[bold red]Application terminated by user.[/bold red]")
            return 1
        
        return 0

# Run the application
if __name__ == "__main__":
    app = RichCLIApp()
    sys.exit(app.run())
```

This comprehensive tutorial covers the major features of the Rich library, from basic text formatting to creating complex command-line interfaces. Rich makes it easy to create beautiful terminal applications with minimal code, and the examples above show how to use it effectively for various tasks.

Would you like to explore any particular aspect of Rich in more detail?