# Instrumentação de aplicações com OpenTelemetry

### Introdução

A implementação de OpenTelemetry em aplicações Go é uma prática essencial para garantir a observabilidade e o monitoramento de desempenho. Ao utilizar o OpenTelemetry, é possível coletar métricas e rastreamentos detalhados sobre o comportamento da aplicação, permitindo uma análise contínua de seu funcionamento e a identificação proativa de problemas. Além disso, a abordagem de Desenvolvimento Orientado a Testes (TDD) foi utilizada para garantir a confiabilidade do código, escrevendo testes automatizados antes do desenvolvimento das funcionalidades. Essa metodologia assegura que a aplicação atenda aos requisitos desde o início, proporcionando um ciclo de desenvolvimento mais seguro e ágil.

### Instalação

Nosso primeiro passo é instalar o SigNoz para que o OpenTelemetry possa enviar os dados para ele. Com o seguinte comando:

```
git clone -b main https://github.com/SigNoz/signoz.git
cd signoz/deploy/
./install.sh
```

Uma vez instalado conseguimos ver a interface inicial do SigNoz pelo http://localhost:3301.

![alt text](<img/image (5).png>)

Seguindo, vamos fazer a instalação do OpenTelemetry SDK, com o comando:

```
go get go.opentelemetry.io/otel/sdk
go get go.opentelemetry.io/otel/exporters/otlp/otlptrace
```

Feito isso vamos para a configuração do rastreamento, que foi desenvolvida com o seguinte código:

```
func initTracer() (*trace.TracerProvider, error) {
    exporter, err := otlptracehttp.New(context.Background())
    if err != nil {
        return nil, err
    }
    resource := resource.NewWithAttributes(
        semconv.SchemaURL,
        semconv.ServiceNameKey.String("your-service-name"),
    )
    tracerProvider := trace.NewTracerProvider(
        trace.WithBatcher(exporter),
        trace.WithResource(resource),
    )
    otel.SetTracerProvider(tracerProvider)
    return tracerProvider, nil
}
```
Configuração das Variáveis de Ambiente: No arquivo main.go, configure as variáveis de ambiente que serão usadas para conectar o OpenTelemetry ao SigNoz:

``` 
export SERVICE_NAME=goGinApp
export OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317
export INSECURE_MODE=true
```

#### Inicialização do OpenTelemetry
Após ca configuração de ambiente, seguimos para a parte do tutorial que é a inicialização do SDK OpenTelemetry. Ao executar o código, as métricas e rastreamentos são configurados conforme esperado:

```
go run main.go
```

### Explicando conceitos e as técnicas do TDD

#### Conceitos do TDD
**Escrita de Testes Antes da Implementação:** Antes de implementar cada funcionalidade, escrevemos testes unitários para definir o comportamento esperado. Isso nos permitiu garantir que o código estivesse correto desde o início.

```
func TestInitTracer(t *testing.T) {
    _, err := initTracer()
    if err != nil {
        t.Fatalf("Falha ao inicializar o Tracer: %v", err)
    }
}
```

**Refatoração com Segurança:** A prática do TDD garante que o código possa ser refatorado sem introduzir novos bugs, pois os testes fornecem um feedback imediato sobre o estado da aplicação.

#### Técnicas de TDD

O TDD é composto por ciclos rápidos de desenvolvimento que envolvem escrever testes automatizados, implementar o código necessário para passar nos testes e refatorar o código para mantê-lo limpo e eficiente.

Test-Driven Development (TDD) é aplicado em cada etapa da implementação, ajudando a criar uma aplicação robusta e confiável desde o início.
Foram utilizados testes como ```TestInitTracer``` para validar a correta inicialização e integração do OpenTelemetry com o SigNoz.

```
func TestInitTracer(t *testing.T) {
    tp, err := initTracer()
    if err != nil {
        t.Fatalf("Erro ao inicializar Tracer: %v", err)
    }
    defer func() { _ = tp.Shutdown(context.Background()) }()
}
```

Esses testes garantem que cada parte do código funcione corretamente antes de ser integrada à base de código principal.

### Conclusão 

A implementação do **OpenTelemetry** em uma aplicação Go, como documentado neste repositório, oferece uma solução eficaz para aprimorar a observabilidade e o monitoramento de serviços. A integração com o **SigNoz** possibilita a visualização e análise de métricas e rastreamentos, facilitando a identificação de gargalos de desempenho e a otimização contínua dos serviços. Seguindo as etapas de configuração e instrumentação descritas, é possível garantir que a aplicação atenda aos requisitos de monitoramento e mantenha um desempenho confiável em produção. A combinação do **OpenTelemetry** com o **SigNoz** representa uma solução acessível e poderosa para equipes que buscam maior controle sobre suas aplicações distribuídas.