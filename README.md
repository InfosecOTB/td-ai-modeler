# AI-Powered Threat Modeling Tool

![td-ai](assets/td-ai-part2.png)

An intelligent threat modeling application that uses Large Language Models (LLMs) to automatically generate security threats and their mitigation proposals for Threat Dragon models.

## Features

- **AI-Powered Threat Generation**: Uses state-of-the-art LLMs to analyze system components and generate comprehensive security threats
- **Threat Framework Support**: Supports STRIDE threat modeling framework, however the code can be adjusted for others as well
- **Multi-LLM Support**: Tested on OpenAI, Anthropic, Google, and Ollama. As the code uses LiteLLM library, it should work with other models as well.
- **Threat Dragon Integration**: Works seamlessly with Threat Dragon JSON models
- **Smart Filtering**: Automatically skips out-of-scope components
- **Data Validation**: Built-in Pydantic validation for threat data integrity
- **Response Validation**: Comprehensive validation of AI responses against original models
- **Validation Logging**: Timestamped validation logs with detailed coverage reports
- **Visual Indicators**: Automatically adds visual cues (red strokes) to components with threats

## Quick Start

### Prerequisites

- Python 3.8+
- API key for your chosen LLM provider

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd td-ai-modeler
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment**
   ```bash
   cp env.example .env
   ```
   
   Edit `.env` with your configuration:
   ```env
   # Choose your LLM provider (uncomment one)
   LLM_MODEL_NAME=openai/gpt-5
   OPENAI_API_KEY=your_openai_api_key_here
   
   # Input files
   INPUT_THREAT_SCHEMA_JSON=owasp.threat-dragon.schema.V2.json
   INPUT_THREAT_MODEL_JSON=your-model.json
   ```

4. **Prepare input files**
   - The Threat Dragon schema file is already in `./input/`
   - Place your threat model JSON file in `./input/`

5. **Run the application**
   ```bash
   python src/main.py
   ```

6. **Check results**
   - Updated model with AI-generated threats will be in `./output/`
   - Validation logs with timestamp will be generated in `./output/logs/`

## Configuration

### Supported LLM Providers

| Provider | Model Example | API Key Variable | Notes |
|----------|---------------|------------------|-------|
| **OpenAI** | `openai/gpt-5` | `OPENAI_API_KEY` | Supports JSON schema validation |
| **Anthropic** | `anthropic/claude-opus-4-1-20250805` | `ANTHROPIC_API_KEY` | High-quality reasoning |
| **Anthropic** | `anthropic/claude-sonnet-4-20250514` | `ANTHROPIC_API_KEY` | Balanced performance |
| **xAI** | `xai/grok-4-latest` | `XAI_API_KEY` | Supports JSON schema validation |
| **xAI** | `xai/grok-4-fast-reasoning-latest` | `XAI_API_KEY` | Supports JSON schema validation  |
| **Google** | `google/gemini-2.5-pro` | `GOOGLE_API_KEY` | Experimental models |
| **Ollama** | `ollama/llama3.2:latest` | None | Local deployment |

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `LLM_MODEL_NAME` | LLM model identifier | `openai/gpt-5` |
| `INPUT_THREAT_SCHEMA_JSON` | Threat Dragon schema filename | `owasp.threat-dragon.schema.V2.json` |
| `INPUT_THREAT_MODEL_JSON` | Input threat model filename | `my-model.json` |

### Advanced Configuration Options

The tool supports several advanced configuration options that can be modified in `src/ai_client.py`:

#### LLM Response Settings
- **`temperature`**: Controls randomness in AI responses (default: 0.1 for consistent results)
- **`max_tokens`**: Maximum tokens in response (default: 24000)
- **`timeout`**: Request timeout in seconds (default: 14400 = 4 hours)

#### JSON Schema Validation
- **`litellm.enable_json_schema_validation`**: Enable structured JSON validation for supported models (OpenAI, xAI)
- **`response_format`**: Force structured JSON response format using Pydantic models

#### Custom API Endpoints
- **`api_base`**: Override default API endpoint for custom deployments or local models
  - Example: `api_base="http://localhost:11434"` for local Ollama
  - Example: `api_base="https://your-custom-endpoint.com"` for custom deployments

#### LiteLLM Configuration
- **`litellm.drop_params`**: Remove unsupported parameters (default: True)
- **`litellm.enable_json_schema_validation`**: Enable JSON schema validation for supported providers

### Configuration Examples

#### Local Ollama Setup
```python
# In src/ai_client.py, uncomment and modify:
# Ollama running on localhost doesn't require api_base to be set
# but if it runs on a remote computer, the parameter has to be adjusted.
api_base="http://server_ip_address:11434"
# LLM_MODEL_NAME=ollama/llama3.2:latest
```

#### Custom API Endpoint
```python
# In src/ai_client.py, uncomment and modify:
api_base="https://your-custom-api.com/v1"
```

#### Enhanced JSON Validation
```python
# In src/ai_client.py, uncomment:
litellm.enable_json_schema_validation = True
response_format = AIThreatsResponseList
```

## Project Structure

```
td-ai-modeler/
├── src/
│   ├── main.py              # Main application entry point
│   ├── ai_client.py         # LLM integration and threat generation
│   ├── utils.py             # File operations and model updates
│   ├── models.py            # Pydantic data models
│   └── validator.py         # AI response validation
├── input/                   # Input files directory
│   ├── owasp.threat-dragon.schema.V2.json
│   └── your-model.json
├── output/                  # Generated output directory
│   └── logs/               # Validation logs
├── prompt.txt               # AI threat modeling prompt template
├── env.example              # Environment configuration template
├── requirements.txt         # Python dependencies
└── README.md               # This file
```

## How It Works

1. **Input Processing**: Loads Threat Dragon schema and model files
2. **AI Threat Generation**: Uses LLM to analyze components and generate threats
3. **Data Validation**: Ensures all generated threats have required fields
4. **Response Validation**: Validates AI response completeness and accuracy
5. **Model Update**: Updates the threat model while preserving original formatting
6. **Visual Updates**: Adds red stroke indicators to components with threats
7. **Validation Logging**: Generates detailed validation reports with timestamps

## Validation Features

The tool includes comprehensive validation to ensure AI responses are complete and accurate:

### Validation Categories
- **INFO**: Elements in scope but missing threats (informational)
- **WARNINGS**: Out-of-scope elements or quality issues (non-blocking)
- **ERRORS**: Completely different IDs with no model overlap (blocking)

### Validation Checks
- **Coverage Validation**: Ensures all in-scope elements (outOfScope=false) have threats
- **ID Validation**: Verifies all response IDs correspond to valid model elements
- **Quality Validation**: Checks that threats include proper mitigation strategies (empty mitigations generate warnings)
- **Data Integrity**: Validates threat structure and required fields

### Validation Outputs
- **Console Summary**: Real-time validation results with coverage statistics
- **Detailed Logs**: Timestamped logs in `./output/logs/` directory
- **Error Reporting**: Specific details about missing elements and invalid IDs
- **Coverage Metrics**: Percentage of in-scope elements with generated threats

### Validation Notes
- Trust boundary boxes and curves are excluded from validation
- Missing elements are informational, not errors
- Invalid IDs (out of scope) are warnings, not errors
- Only completely different IDs are validation errors

Validation runs automatically during threat generation and creates detailed logs in the `./output/logs/` directory.

## Troubleshooting

### Common Issues

#### LLM Response Errors
- **Invalid JSON**: The tool automatically attempts to extract JSON from malformed responses
- **Timeout Issues**: Increase `timeout` value in `ai_client.py` for large models
- **Token Limits**: Adjust `max_tokens` based on model capabilities

#### Validation Warnings
- **Missing Elements**: Normal for complex models - elements may be out of scope
- **Empty Mitigations**: Check AI response quality or adjust prompt template
- **Out-of-Scope Elements**: Elements not in scope but have threats generated
- **Invalid IDs**: Verify model structure and element IDs

#### Configuration Issues
- **API Key Errors**: Ensure correct environment variables are set
- **Model Not Found**: Verify model name format matches provider requirements
- **Connection Issues**: Check `api_base` URL for custom endpoints

### Performance Optimization

#### For Large Models
- Increase `timeout` to 28800 (8 hours) for very large threat models
- Use `max_tokens=32000` for models with higher token limits
- Consider using faster models for initial threat generation

#### For Local Models (Ollama)
- Ensure sufficient hardware (GPU, CPU, RAM)
- Monitor system resources during generation

## Development

### Running the Application

```bash
# Install development dependencies
pip install -r requirements.txt

# Run the application
python src/main.py
```

### Code Structure

- **`main.py`**: Orchestrates the entire threat modeling process
- **`ai_client.py`**: Handles LLM communication and threat generation
- **`utils.py`**: File operations and model manipulation utilities
- **`models.py`**: Pydantic models for threat data validation
- **`validator.py`**: Comprehensive validation of AI responses

### Customization

#### Modifying the AI Prompt
Edit `prompt.txt` to customize threat generation behavior:
- Add specific threat frameworks
- Modify threat categories
- Adjust output format requirements

#### Adding New LLM Providers
1. Add provider configuration to `env.example`
2. Update provider table in README
3. Test with sample threat model


## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Threat Dragon](https://threatdragon.org/) for the excellent threat modeling framework
- [LiteLLM](https://github.com/BerriAI/litellm) for seamless multi-LLM support
- [Pydantic](https://pydantic.dev/) for robust data validation

## Support

If you encounter any issues or have questions:

1. Check the [Issues](https://github.com/your-username/td-ai-modeler/issues) page
2. Create a new issue with detailed information
3. Include your configuration and error messages

---

**Built for security professionals and threat modeling practitioners**