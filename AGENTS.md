# Agent Instructions for Frobyte BYOA

## Role
You are an AI integration agent for the Frobyte Platform. The user has provided this repository to onboard a custom API using the Model Context Protocol (MCP). Your task is to read this schema and generate a working MCP server wrapper.

## API Context
This repository defines a "Customer Support" legacy API. It is used to look up customer balances and subscription statuses.

## Integration Rules & Guardrails
When generating the MCP tools for this API, you MUST follow these strict rules:
1. **Never hardcode credentials:** The API requires an `Authorization: Bearer <token>` header. You must configure the MCP server to expect this token via the Frobyte OAuth Token Exchange (RFC 8693) mechanism.
2. **Data Minimization:** When returning customer data, strip out any Personally Identifiable Information (PII) such as `social_security_number` or `home_address` before returning the payload to the LLM context window.
3. **Pagination:** If a query returns more than 10 records, you must implement pagination in the tool to prevent MCP token bloat.
4. **Error Handling:** If the API returns a 404, the tool must return a graceful JSON response: `{"status": "error", "message": "Customer not found"}`, do not crash the MCP server.

## Target Schema
Refer to the `openapi.yaml` file in this repository for the exact endpoint structures and parameters needed to generate the tool input schemas.

--------------------------------------------------------------------------------
File 3: openapi.yaml (The Technical Blueprint)
Developers often bring OpenAPI specifications. Your platform can use this to automatically generate the MCP tools and parameters
.
File Name: openapi.yaml (in the root of your repo)
openapi: 3.0.0
info:
  title: Mock Customer Support API
  description: A mock API to test Frobyte's auto-discovery and MCP wrapping.
  version: 1.0.0
servers:
  - url: https://api.example-demo.com/v1
paths:
  /customers/{customer_id}:
    get:
      summary: Get customer details
      operationId: get_customer_data
      parameters:
        - name: customer_id
          in: path
          required: true
          schema:
            type: string
          description: The ID of the customer to retrieve.
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: string
                  name:
                    type: string
                  status:
                    type: string
                    example: "Active"
                  balance_kes:
                    type: integer
                    example: 4500
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
security:
  - bearerAuth: []