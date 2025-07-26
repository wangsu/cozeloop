# Coze Loop User Tutorial: Complete Guide to AI Agent Development

Welcome to Coze Loop! This comprehensive tutorial will guide you step-by-step through using Coze Loop for AI agent development, from prompt creation to evaluation and monitoring.

## 🎯 What You'll Learn

- How to develop and debug prompts with the visual Playground
- Create and manage evaluation datasets and evaluators
- Run comprehensive experiments to test your AI agents
- Monitor and trace your AI agent's performance in production
- Best practices for the complete AI agent development lifecycle

---

## 📋 Prerequisites

Before starting, ensure you have:
- Coze Loop installed and running (see [Quick Start](README.md#quickstart))
- Access to the web interface at `http://localhost:8082`
- At least one LLM model configured (OpenAI, Claude, etc.)
- Basic understanding of prompt engineering concepts

---

## 🚀 Getting Started

### Step 1: Access Your Workspace

1. **Open Coze Loop**
   - Navigate to `http://localhost:8082` in your browser
   - Log in with your credentials

2. **Create or Select a Space**
   - Spaces are workspaces that contain all your prompts, datasets, and experiments
   - Click "Create Space" for a new project or select an existing one
   - Spaces can be Personal (just you) or Team (shared with colleagues)

**What happens behind the scenes:**
- Your login session is stored in the `user` table
- Space membership is managed through `space_user` table with role-based access
- All subsequent resources will be associated with your selected space

---

## 🎨 Part 1: Prompt Development & Debugging

### Creating Your First Prompt

**Step 1: Navigate to Prompt Development**
1. In your space, click on "Prompts" in the sidebar
2. Click "Create New Prompt"
3. Fill in the basic information:
   - **Name**: Give your prompt a descriptive name (e.g., "Customer Support Bot")
   - **Description**: Explain what this prompt does
   - **Key**: Unique identifier for API access

**Step 2: Design Your Prompt**
1. **Choose Template Type**: 
   - Normal: Standard text-based prompts
   - Chat: Multi-turn conversation prompts

2. **Write Your Messages**:
   ```
   System: You are a helpful customer support assistant. Always be polite and provide accurate information.
   
   User: {{customer_query}}
   ```

3. **Define Variables**:
   - Click "Add Variable"
   - Name: `customer_query`
   - Type: `string`
   - Description: "The customer's question or issue"

4. **Configure Model Settings**:
   - Model: Choose your LLM (e.g., GPT-4, Claude)
   - Temperature: 0.7 (for balanced creativity)
   - Max Tokens: 500
   - Top-p: 0.9

**What happens behind the scenes:**
- Prompt metadata is stored in `prompt_basic` table
- Your draft is automatically saved in `prompt_user_draft` table
- Variable definitions are stored as JSON in the draft

### Using the Playground for Testing

**Step 3: Test Your Prompt**
1. **Access the Playground**:
   - Click "Playground" tab in your prompt editor
   - This is your interactive testing environment

2. **Set Variable Values**:
   - In the Variables panel, enter test values:
   - `customer_query`: "I can't login to my account"

3. **Run the Test**:
   - Click "Send" to execute the prompt
   - Watch the real-time response generation
   - Response time, token usage, and costs are tracked

4. **Analyze Results**:
   - Review the output quality
   - Check response time (shown in milliseconds)
   - Monitor token consumption

**Step 4: Compare Models**
1. **Split View Testing**:
   - Click "Compare Models"
   - Select 2-3 different models (e.g., GPT-4, Claude, Local model)
   - Send the same input to all models simultaneously

2. **Evaluate Differences**:
   - Compare response quality, style, and accuracy
   - Note performance differences (speed, cost)
   - Choose the best model for your use case

**What happens behind the scenes:**
- Each test execution creates a `prompt_debug_log` entry
- Model requests are tracked in `model_request_record` table
- Debug context (variables, configs) is stored in `prompt_debug_context`
- Real-time traces are sent to ClickHouse for observability

### Version Management

**Step 5: Commit Your Prompt**
1. **When Ready to Save**:
   - Click "Commit Version"
   - Enter version description: "Initial customer support prompt v1.0"
   - Click "Commit"

2. **Version History**:
   - View all versions in the "Versions" tab
   - Compare different versions side-by-side
   - Revert to previous versions if needed

**What happens behind the scenes:**
- A new record is created in `prompt_commit` table
- Version number is auto-generated and stored
- `prompt_basic.latest_version` is updated
- Full prompt configuration (messages, model config, tools) is preserved

---

## 📊 Part 2: Dataset Management

### Creating Evaluation Datasets

**Step 1: Create a Dataset**
1. **Navigate to Datasets**:
   - Click "Datasets" in the sidebar
   - Click "Create Dataset"

2. **Configure Dataset**:
   - **Name**: "Customer Support Test Cases"
   - **Category**: "evaluation" 
   - **Description**: "Test cases for customer support prompt"

**Step 2: Define Data Schema**
1. **Schema Design**:
   - Click "Manage Schema"
   - Add fields that match your prompt variables:
   ```json
   {
     "customer_query": {
       "type": "string",
       "description": "Customer's question"
     },
     "expected_response": {
       "type": "string", 
       "description": "Ideal response"
     },
     "category": {
       "type": "string",
       "description": "Issue category"
     }
   }
   ```

**Step 3: Add Test Data**
1. **Manual Entry**:
   - Click "Add Item"
   - Fill in your test cases:
   
   | customer_query | expected_response | category |
   |---|---|---|
   | "I forgot my password" | "I can help you reset your password. Please..." | "authentication" |
   | "My order hasn't arrived" | "Let me check your order status..." | "shipping" |
   | "I want to cancel my subscription" | "I understand you'd like to cancel..." | "billing" |

2. **Bulk Import**:
   - Click "Import" → "Upload CSV/JSON"
   - Select your prepared data file
   - Map columns to schema fields
   - Click "Import Data"

**Step 4: Create Dataset Version**
1. **Version Your Data**:
   - Click "Create Version"
   - Name: "v1.0-initial-test-cases"
   - Description: "50 customer support test scenarios"
   - Click "Create"

**What happens behind the scenes:**
- Dataset metadata stored in `dataset` table
- Schema definition saved in `dataset_schema` table  
- Individual items stored in `dataset_item` table
- Version snapshot created in `dataset_version` and `dataset_item_snapshot` tables
- Import jobs tracked in `dataset_io_job` table

---

## 🔍 Part 3: Evaluation System

### Creating Evaluators

**Step 1: Choose Evaluator Type**
1. **Navigate to Evaluators**:
   - Click "Evaluators" in the sidebar
   - Click "Create Evaluator"

2. **Select Built-in Templates**:
   - **Relevance**: Does the response address the question?
   - **Correctness**: Is the information factually accurate?
   - **Helpfulness**: Is the response useful to the user?
   - **Conciseness**: Is the response appropriately brief?

**Step 2: Configure Custom Evaluator**
1. **Create Custom Evaluator**:
   - Choose "Custom Prompt Evaluator"
   - Name: "Customer Satisfaction Evaluator"

2. **Write Evaluation Prompt**:
   ```
   You are evaluating customer support responses for satisfaction.
   
   Rate the response on customer satisfaction (0.0 to 1.0):
   - 1.0: Extremely helpful, polite, and resolves the issue
   - 0.5: Somewhat helpful but could be improved  
   - 0.0: Unhelpful, rude, or doesn't address the issue
   
   Customer Query: {{input}}
   Support Response: {{output}}
   Expected Response: {{reference_output}}
   
   Provide your score and reasoning.
   ```

3. **Configure Evaluation Model**:
   - Model: GPT-4 (for consistent evaluation)
   - Temperature: 0.1 (for consistent scoring)
   - Output Format: JSON with score and reason

**What happens behind the scenes:**
- Evaluator config stored in `evaluator` table
- Version history maintained in `evaluator_version` table
- Built-in evaluator templates loaded from `evaluation.yaml` config
- Custom prompts stored with input/output schema definitions

### Creating Evaluation Targets

**Step 1: Define What to Evaluate**
1. **Navigate to Targets**:
   - Click "Evaluation Targets"
   - Click "Create Target"

2. **Target Configuration**:
   - **Type**: "Prompt" (evaluating your customer support prompt)
   - **Source**: Select your customer support prompt
   - **Version**: Choose specific version or "latest"

**What happens behind the scenes:**
- Target metadata stored in `eval_target` table
- Links to source prompt via `source_target_id`
- Version tracking in `eval_target_version` table

---

## 🧪 Part 4: Running Experiments

### Setting Up Your First Experiment

**Step 1: Create Experiment**
1. **Navigate to Experiments**:
   - Click "Experiments" in the sidebar
   - Click "Create Experiment"

2. **Experiment Configuration**:
   - **Name**: "Customer Support Prompt v1.0 Evaluation"
   - **Description**: "Testing initial customer support prompt against test dataset"
   - **Evaluation Dataset**: Select your customer support dataset v1.0
   - **Target**: Select your customer support prompt target
   - **Evaluators**: Choose 3-4 evaluators:
     - Relevance
     - Helpfulness  
     - Correctness
     - Your custom Customer Satisfaction evaluator

**Step 2: Configure Execution Settings**
1. **Evaluation Configuration**:
   - **Sampling**: All items (for complete evaluation)
   - **Parallel Execution**: 5 concurrent evaluations
   - **Timeout**: 60 seconds per item
   - **Retry**: 3 attempts for failed evaluations

**Step 3: Launch Experiment**
1. **Start Execution**:
   - Review your configuration
   - Click "Start Experiment"
   - Monitor progress in real-time

**Step 4: Monitor Progress**
1. **Live Updates**:
   - Watch the progress bar showing completion percentage
   - See successful vs failed evaluations
   - View estimated time to completion
   - Monitor resource usage (tokens, costs)

**What happens behind the scenes:**
- Experiment metadata stored in `experiment` table
- Execution tracked in `expt_run_log` table
- Individual item results stored in `expt_item_result` table
- Turn-by-turn results in `expt_turn_result` table
- Aggregated metrics calculated and stored in `expt_aggr_result` table
- Background workers process evaluation queue via RocketMQ
- Progress updates pushed via WebSocket connections

### Analyzing Results

**Step 5: Review Experiment Results**
1. **Overall Metrics Dashboard**:
   - Average scores across all evaluators
   - Score distribution histograms
   - Pass/fail rates for each criterion
   - Performance statistics (latency, token usage)

2. **Detailed Analysis**:
   - **By Evaluator**: See how your prompt performed on each dimension
     - Relevance: 0.85 average (good)
     - Helpfulness: 0.78 average (needs improvement)
     - Correctness: 0.92 average (excellent)
     - Customer Satisfaction: 0.73 average (fair)

3. **Item-Level Results**:
   - Click into individual test cases
   - See actual vs expected outputs
   - Review evaluator reasoning
   - Identify patterns in failures

**Step 6: Identify Improvement Areas**
1. **Error Analysis**:
   - Filter for low-scoring items
   - Group by failure patterns:
     - Authentication queries: Lower helpfulness scores
     - Billing questions: Higher customer satisfaction
     - Technical issues: Mixed results

2. **Export Results**:
   - Download detailed CSV report
   - Export for further analysis in external tools
   - Share results with team members

**What happens behind the scenes:**
- Results aggregated from multiple database tables
- Statistics calculated in real-time from `expt_stats` table
- Data export functionality queries across normalized tables
- Charts and visualizations rendered from aggregated metrics

---

## 📈 Part 5: Observability & Monitoring

### Setting Up Production Monitoring

**Step 1: Deploy Your Prompt**
1. **API Integration**:
   - Get your prompt's API endpoint
   - Copy the API key from your settings
   - Integrate into your application:
   ```python
   import requests
   
   response = requests.post(
       "http://localhost:8080/api/prompt/execute",
       headers={"Authorization": "Bearer YOUR_API_KEY"},
       json={
           "prompt_key": "customer-support-bot",
           "variables": {
               "customer_query": "I need help with my order"
           }
       }
   )
   ```

**Step 2: Enable Tracing**
1. **SDK Integration**:
   - Install Coze Loop SDK in your application
   - Configure automatic trace reporting:
   ```python
   from cozeloop import CozeLoopTracer
   
   tracer = CozeLoopTracer(
       endpoint="http://localhost:8080",
       api_key="YOUR_API_KEY"
   )
   
   # Automatic tracing of all prompt executions
   tracer.start()
   ```

**Step 3: Monitor in Real-time**
1. **Access Observability Dashboard**:
   - Click "Observability" in sidebar
   - View real-time metrics:
     - Request volume and trends
     - Response times (P50, P95, P99)
     - Success/error rates
     - Token consumption and costs

2. **Trace Analysis**:
   - **Request Tracing**: See the full journey of each request
     - User input → Prompt processing → Model call → Response
   - **Performance Metrics**: Identify bottlenecks
     - Time to first token
     - Total response time
     - Model processing time
   - **Error Investigation**: Debug failed requests
     - Stack traces for errors
     - Input/output context
     - Model response details

**What happens behind the scenes:**
- All API calls automatically generate traces
- Trace data flows through RocketMQ to ClickHouse
- Real-time metrics calculated from `observability_spans` table
- Dashboards query ClickHouse for high-performance analytics
- Error traces capture full request context for debugging

### Creating Custom Views and Alerts

**Step 4: Set Up Monitoring Views**
1. **Create Custom Views**:
   - Filter for high-latency requests (>10 seconds)
   - Monitor error patterns by user or query type
   - Track token usage by time period

2. **Set Up Alerts** (if configured):
   - High error rate (>5% in 5 minutes)
   - Unusual latency spikes
   - Excessive token consumption

**What happens behind the scenes:**
- Custom views stored as filter configurations
- Real-time queries against trace data
- Alert conditions evaluated continuously
- Notifications sent via configured channels

---

## 🔄 Part 6: Iterative Improvement Workflow

### The Complete Development Cycle

**Phase 1: Iterate Based on Results**
1. **Analyze Experiment Results**:
   - Low helpfulness scores identified
   - Authentication queries need improvement
   - Billing responses too generic

2. **Modify Your Prompt**:
   - Go back to Prompt Development
   - Create new version with improvements:
   ```
   System: You are a helpful customer support assistant. Always be polite and provide accurate, specific information.
   
   For authentication issues:
   - Provide step-by-step instructions
   - Offer multiple recovery options
   - Include security best practices
   
   For billing questions:
   - Ask clarifying questions about specific charges
   - Explain billing cycles clearly
   - Provide contact information for complex issues
   
   User: {{customer_query}}
   ```

3. **Test in Playground**:
   - Use the same test cases that failed before
   - Verify improvements in response quality
   - Test edge cases and corner scenarios

**Phase 2: Re-evaluate**
1. **Run New Experiment**:
   - Same dataset, same evaluators
   - Compare v1.0 vs v2.0 results
   - Look for measurable improvements

2. **A/B Testing**:
   - Deploy both versions to production
   - Split traffic 50/50
   - Monitor real-world performance differences

**Phase 3: Production Monitoring**
1. **Track Key Metrics**:
   - Customer satisfaction scores
   - Resolution time improvements  
   - Escalation rate reductions
   - Cost per interaction

2. **Continuous Optimization**:
   - Weekly performance reviews
   - Monthly model updates
   - Quarterly comprehensive evaluations

---

## 🛠 Advanced Features

### Working with Tools and Function Calling

**Step 1: Add Tools to Your Prompt**
1. **Define Tools**:
   ```json
   {
     "name": "lookup_order",
     "description": "Look up customer order information",
     "parameters": {
       "type": "object",
       "properties": {
         "order_id": {"type": "string"},
         "email": {"type": "string"}
       }
     }
   }
   ```

2. **Configure Tool Calling**:
   - Enable function calling in model config
   - Set tool choice to "auto"
   - Test tool integration in playground

### Multi-turn Conversation Evaluation

**Step 1: Create Conversation Datasets**:
- Design multi-turn scenarios
- Include conversation context
- Evaluate conversational flow and coherence

**Step 2: Advanced Evaluators**:
- Context maintenance evaluation
- Conversation coherence scoring
- Multi-turn helpfulness assessment

### Custom Model Integration

**Step 1: Add Your Model**:
1. Configure in `model_config.yaml`
2. Test connectivity in playground
3. Compare performance against existing models

---

## 🎓 Best Practices

### Prompt Development
- **Start Simple**: Begin with basic prompts, add complexity gradually
- **Use Variables**: Make prompts reusable with well-defined variables
- **Version Everything**: Always commit versions with clear descriptions
- **Test Thoroughly**: Use diverse test cases in playground

### Dataset Management
- **Quality over Quantity**: Better to have 50 high-quality test cases than 500 poor ones
- **Represent Real Usage**: Include actual user queries and edge cases
- **Version Your Data**: Track dataset changes over time
- **Regular Updates**: Add new test cases based on production issues

### Evaluation Strategy
- **Multiple Dimensions**: Don't rely on a single evaluator
- **Consistent Standards**: Use the same evaluation criteria across experiments
- **Human Validation**: Periodically validate evaluator accuracy with human review
- **Baseline Comparisons**: Always compare against previous versions

### Production Monitoring
- **Monitor Key Metrics**: Focus on metrics that matter to your business
- **Set Meaningful Alerts**: Avoid alert fatigue with well-tuned thresholds
- **Regular Reviews**: Schedule weekly/monthly performance reviews
- **Cost Tracking**: Monitor token usage and costs continuously

---

## 🔧 Troubleshooting

### Common Issues

**Prompt Execution Fails**
- Check model configuration and API keys
- Verify variable definitions match usage
- Review error logs in observability dashboard

**Low Evaluation Scores** 
- Review evaluator prompts for clarity
- Check if test data represents real usage
- Validate expected outputs are reasonable

**Performance Issues**
- Check ClickHouse and database connectivity
- Monitor resource usage (CPU, memory)
- Review RocketMQ queue lengths

**Data Import Problems**
- Verify CSV/JSON format matches schema
- Check for encoding issues (use UTF-8)
- Validate required fields are present

### Getting Help
- Check system logs: `docker logs cozeloop-app`
- Review configuration files in `conf/default/`
- Monitor database connectivity and health
- Check the [troubleshooting guide](https://github.com/coze-dev/CozeLoop/wiki/7.-Troubleshooting)

---

## 🎉 Conclusion

Congratulations! You now have a comprehensive understanding of Coze Loop's capabilities:

✅ **Prompt Development**: Create, test, and iterate on prompts using the visual playground
✅ **Dataset Management**: Build and maintain high-quality evaluation datasets  
✅ **Evaluation System**: Run comprehensive experiments with multiple evaluators
✅ **Production Monitoring**: Track performance and debug issues in real-time
✅ **Iterative Improvement**: Use data-driven insights to continuously improve your AI agents

### Next Steps
1. **Start Your First Project**: Create a space and begin with a simple prompt
2. **Build Your Dataset**: Collect real user queries for evaluation
3. **Run Regular Experiments**: Establish a weekly evaluation schedule
4. **Monitor Production**: Set up observability for your deployed prompts
5. **Join the Community**: Connect with other Coze Loop users for tips and best practices

### Additional Resources
- **API Documentation**: Complete API reference for integration
- **SDK Examples**: Sample code in Python, JavaScript, and Go
- **Community Forum**: Ask questions and share experiences
- **Video Tutorials**: Step-by-step video guides for complex workflows

Happy AI agent development! 🚀