# Using Newman with CSV Data Files - A Tutorial

## Introduction
Newman is Postman's command-line collection runner that allows you to run and test your API collections from the command line. This tutorial explains how to install Newman, then use it with CSV data files to run multiple iterations of your API tests with different data sets.

## Prerequisites
- Node.js installed  
- A Postman collection exported as JSON  
- Basic understanding of Postman and API testing  

## Step 1: Install Node.js
Newman requires Node.js and npm (Node Package Manager).  

1. Download Node.js from [https://nodejs.org](https://nodejs.org)  
2. Install it using your platform’s installer  
3. Verify installation:  

    node -v  
    npm -v  

This should print the installed versions.

## Step 2: Install Newman
Once Node.js and npm are installed, you can install Newman globally:

    npm install -g newman

Verify Newman installation:

    newman -v

You should see the installed Newman version number.

## Step 3: Setting Up Your CSV Data File

1. Create a CSV file (e.g., `test-data.csv`) with your test data. The column headers should match the variables you'll use in your Postman requests.

Example `test-data.csv`:

    username,password,email
    user1,pass1,user1@email.com
    user2,pass2,user2@email.com
    user3,pass3,user3@email.com

## Step 4: Preparing Your Postman Collection

1. In your Postman requests, use variables that match your CSV column names  
2. Reference these variables in your requests using double curly braces:

Example request body:

    {
      "username": "{{username}}",
      "password": "{{password}}",
      "email": "{{email}}"
    }

3. Export your collection as a JSON file from Postman.

## Step 5: Running Newman with CSV Data

### Basic Command
The simplest way to run Newman with a CSV file:

    newman run YourCollection.json -d test-data.csv

### Comprehensive Command
A more detailed command with common options:

    newman run YourCollection.json \
      --environment YourEnvironment.json \
      --iteration-data test-data.csv \
      --reporters cli,json,html \
      --reporter-html-export results.html \
      --iteration-count 2 \
      --delay-request 1000 \
      --timeout-request 5000

## Step 6: Understanding Command Options

- `--environment`: Specify your environment file  
- `--iteration-data`: Your CSV data file  
- `--reporters`: Output format (cli, json, html)  
- `--reporter-html-export`: Save results to HTML file  
- `--iteration-count`: Limit number of iterations  
- `--delay-request`: Delay between requests (milliseconds)  
- `--timeout-request`: Request timeout (milliseconds)  

## Best Practices

1. **CSV Structure**
   - Ensure CSV column headers exactly match your variable names  
   - Use consistent data formatting  
   - Include all required variables  

2. **Error Handling**
   - Add appropriate test cases in your collection  
   - Use pre-request scripts for data validation  
   - Check Newman's exit codes for automation  

3. **Performance**
   - Use `--delay-request` to prevent rate limiting  
   - Monitor response times  
   - Consider parallel execution for large datasets  

## Common Issues and Solutions

### Issue 1: Variables Not Being Replaced
**Solution**: Verify that CSV column names exactly match the variable names in your requests  

### Issue 2: Newman Exits Unexpectedly
**Solution**: Check for:
- Valid JSON in collection file  
- Correct file paths  
- CSV file encoding (use UTF-8)  

### Issue 3: Rate Limiting
**Solution**: Add delays between requests:

    newman run collection.json -d data.csv --delay-request 1000

## Example Complete Workflow

1. Install Newman via npm:

    npm install -g newman

2. Create your CSV file:

    username,password,email
    user1,pass1,user1@email.com
    user2,pass2,user2@email.com

3. Export your Postman collection  

4. Run Newman with reporting:

    newman run MyCollection.json \
      --environment MyEnvironment.json \
      --iteration-data test-data.csv \
      --reporters cli,html \
      --reporter-html-export results.html

5. Check the generated reports in your specified output location  

## Conclusion
Newman with CSV data files provides a powerful way to automate API testing with multiple data sets. By following this tutorial, you can set up Newman, prepare test data, and run robust automated API tests that can be integrated into your CI/CD pipeline.

## Additional Resources
- [Newman Documentation](https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman/)  
- [Postman Learning Center](https://learning.postman.com/)  
- [Newman GitHub Repository](https://github.com/postmanlabs/newman)  