# 🧰 Makefile para Proyecto Go – Módulos Ejemplo con Passkeys
# Este Makefile define comandos estándar para trabajar con los módulos ejemplo.

APP_NAME=eav-api
PKG_PATH=./...

.PHONY: all test lint format run doc exampleuser examplelogin clean

all: lint test

## 🔍 Ejecuta linter completo con golangci-lint
lint:
	@echo "Ejecutando linter..."
	@golangci-lint run

## 🎯 Ejecuta pruebas de todos los paquetes
test:
	@echo "Ejecutando pruebas..."
	@go test $(PKG_PATH) -v -cover

## 🧹 Formatea imports y código
format:
	@echo "Formateando código..."
	@goimports -w .

## 🚀 Ejecuta el servicio principal (cmd/eav-api/main.go)
run:
	@echo "Iniciando $(APP_NAME)..."
	@go run ./cmd/$(APP_NAME)/main.go

## 📚 Genera documentación con godoc (requiere instalación local)
doc:
	@echo "Sirviendo documentación en http://localhost:6060"
	@godoc -http=:6060

## 🧪 Ejecuta pruebas del módulo exampleuser
exampleuser:
	@go test ./internal/exampleuser/... -v

## 🧪 Ejecuta pruebas del módulo examplelogin
examplelogin:
	@go test ./internal/examplelogin/... -v

## 🧹 Limpieza general (borrar binarios, cache, etc.)
clean:
	@echo "Limpiando artefactos temporales..."
	@go clean
	@rm -rf dist/
