# n8n Local with Weaviate Vector Database

A complete Docker-based setup for running n8n workflow automation with Weaviate vector database for building RAG (Retrieval-Augmented Generation) applications locally.

## 📋 Overview

This repository provides a production-ready Docker Compose configuration for:
- **n8n**: Workflow automation platform for building AI agents and automations
- **Weaviate**: AI-native vector database for semantic search and RAG
- **PostgreSQL**: Database backend for n8n data persistence

Perfect for building AI-powered chatbots, document Q&A systems, and knowledge bases with vector search capabilities.

## 🏗️ Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   n8n       │────▶│  PostgreSQL  │     │  Weaviate   │
│  (Port 5678)│     │              │     │ (Port 8080) │
└─────────────┘     └──────────────┘     └─────────────┘
       │                                         │
       └─────────────────────────────────────────┘
              Docker Network: n8n-weaviate
```

## 🚀 Quick Start

### Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- 4GB+ RAM available
- 10GB+ disk space

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/CHARIFA10/n8n_local.git
   cd n8n_local
   ```

2. **Configure environment variables**
   Edit `.env` file with your settings:
   ```bash
   # PostgreSQL Configuration
   POSTGRES_USER=n8n_user
   POSTGRES_PASSWORD=your_secure_password_here
   POSTGRES_DB=n8n_db
   POSTGRES_NON_ROOT_USER=n8n_app
   POSTGRES_NON_ROOT_PASSWORD=your_app_password_here
   ```

3. **Start the services**
   ```bash
   docker-compose up -d
   ```

4. **Verify services are running**
   ```bash
   docker-compose ps
   ```

56. **Access the applications**
   - **n8n**: http://localhost:5678
   - **Weaviate**: http://localhost:8080
   - **Weaviate API**: http://localhost:8080/v1

## 📁 Project Structure

```
n8n_local/
├── docker-compose.yml       # Main Docker Compose configuration
├── .env                     # Environment variables 
├── init-data.sh            # PostgreSQL initialization script
├── documents/              # Mount point for documents to process
└── README.md              # This file
```

## 🔧 Configuration

### Docker Volumes

The setup uses three persistent volumes:

- **db_storage**: PostgreSQL data
- **n8n_storage**: n8n workflows and credentials
- **weaviate_data**: Weaviate vector embeddings and indices

### Ports

- `5678`: n8n web interface
- `8080`: Weaviate REST API
- `50051`: Weaviate gRPC (for high-performance queries)

### Network

All services communicate via the `n8n-weaviate` Docker network for secure internal communication.

## 🎯 Use Cases

### 1. Document Q&A System (RAG)

Build an AI assistant that answers questions based on your documents:
- Upload PDFs, DOCX, or text files
- Automatically chunk and embed documents
- Query with natural language
- Get contextual answers with source citations

### 2. HR Policy Assistant

Create a chatbot that helps employees understand company policies:
- Index policy documents in Weaviate
- Natural language queries about leave, conduct, benefits
- Always up-to-date with latest policies

### 3. Knowledge Base Search

Semantic search across your organization's knowledge:
- Index wikis, documentation, tickets
- Find relevant information even with different wording
- Reduce time spent searching for information

## 📚 Example Workflows

### Basic RAG Workflow

1. **Document Ingestion**
   - Read files from `/documents` folder
   - Extract text content
   - Chunk into manageable pieces
   - Generate embeddings
   - Store in Weaviate

2. **Query & Response**
   - Receive user question
   - Generate query embedding
   - Search Weaviate for relevant chunks
   - Pass context to LLM
   - Return AI-generated answer

## 🛠️ Common Operations

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### Stop Services and Remove Data
```bash
docker-compose down -v
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f weaviate
```

### Restart a Service
```bash
docker-compose restart n8n
```

### Check Service Status
```bash
docker-compose ps
```

## 🔍 Weaviate Operations

### Check Weaviate Health
```bash
curl http://localhost:8080/v1/.well-known/ready
```

### View Weaviate Schema
```bash
curl http://localhost:8080/v1/schema
```

### Query Weaviate Objects
```bash
curl http://localhost:8080/v1/objects
```

## 🐛 Troubleshooting

### n8n Cannot Connect to Weaviate

**Error**: `ENOTFOUND weaviate`

**Solution**:
```bash
docker-compose down
docker-compose up -d
```

Verify network:
```bash
docker network inspect n8n-weaviate
```

### Weaviate Data Not Persisting

**Check volume exists**:
```bash
docker volume ls | grep weaviate
```

**Inspect volume**:
```bash
docker volume inspect n8n_local_weaviate_data
```

### PostgreSQL Connection Issues

**Check credentials in `.env`** match those in docker-compose.yml

**View PostgreSQL logs**:
```bash
docker-compose logs postgres
```

### Out of Memory

Weaviate and vector operations are memory-intensive. Ensure:
- Docker has at least 4GB RAM allocated
- Reduce vector dimensions if needed
- Limit batch sizes during ingestion

### Port Already in Use

If ports 5678 or 8080 are taken:

Edit `docker-compose.yml`:
```yaml
ports:
  - "5679:5678"  # Change host port
```

## 🔐 Security Considerations

### For Production Use:

1. **Change Default Passwords**
   - Use strong, unique passwords in `.env`
   - Never commit `.env` to version control

2. **Enable Authentication**
   Update Weaviate config:
   ```yaml
   environment:
     AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'false'
     AUTHENTICATION_APIKEY_ENABLED: 'true'
     AUTHENTICATION_APIKEY_ALLOWED_KEYS: 'your-secret-key'
   ```

3. **Use HTTPS**
   - Add reverse proxy (Nginx/Traefik)
   - Obtain SSL certificates



## 📊 Performance Tuning

### Weaviate Performance

For better performance with large datasets:

```yaml
weaviate:
  environment:
    QUERY_MAXIMUM_RESULTS: 10000
    LIMIT_RESOURCES: 'false'
  deploy:
    resources:
      limits:
        memory: 8G
      reservations:
        memory: 4G
```

### n8n Performance

For handling many workflows:

```yaml
n8n:
  environment:
    - EXECUTIONS_MODE=queue
    - N8N_METRICS=true
```

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 Resources

### Documentation
- [n8n Documentation](https://docs.n8n.io/)
- [Weaviate Documentation](https://weaviate.io/developers/weaviate)
- [Docker Compose Reference](https://docs.docker.com/compose/)

### Tutorials
- [Building RAG Applications with n8n and Weaviate](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreweaviate/)
- [Weaviate Quick Start](https://weaviate.io/developers/weaviate/quickstart)

### Community
- [n8n Community Forum](https://community.n8n.io/)
- [Weaviate Slack](https://weaviate.io/slack)

## 🙏 Acknowledgments

- n8n team for the amazing workflow automation platform
- Weaviate team for the powerful vector database
- Open source community for making this possible

## 📧 Support

For issues and questions:
- Open an issue on [GitHub](https://github.com/CHARIFA10/n8n_local/issues)
- Check existing issues for solutions

---

**Built with ❤️ for the AI automation community**