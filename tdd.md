# TDD и все, что с ним связано.
#### Для чего нужен ####
Есть несколько причин, для внедрения TDD. Некоторые преимущества от внедрения TDD:
- Код становится самодокументированным. Документация в виде тестов обычно самая актуальная и достаточно подробная.
- Подразумевается, что приложение всегда находится в рабочем состоянии, просто с различной полнотой реализиации указанных требований. 
- Само написание когда в такой методологии заставляет писать приложение небольшими частями. 

#### Порядок работы. 
- пишется тест для нужного функционала
- пишется код, который проходит этот тест
- проводится рефакторинг написанного кода.

#### Типы тестов.
- Модульное тестирование - тестируются отдельные модули, как самостоятельные единицы.
- Интеграционные тесты - позволяют определить как уже протестированные модули взаимодействуют друг-с-другом.

#### Стек технологий для тестирования.
- xUnit, MS Unit Testing Framework, и пр.  Разница между ними невелика. Служат для тестирования модулей приложения.
- stub - это скорее заглушки для реально существующих объектов, не содержат логики.
- Mock - это более интеллектуальная заглушка, можно указать поведение объекта, например что этот метод должен быть вызван более двух раз.

#### Какой код обычно тестируется Юнит тестами?
- в первую очередь тестируются нетривиальные участки кода.
- участки кода которые часто изменяются.
- участки кода критически высокой важности.
- участки высокой связности.
## Примеры использования Moq библиотеки.
Библиотека Moq не делает особой разницы между **stub** и **mock**
Рассмотрим интерфейс, котороый нужно будет протестировать
```C#
public interface ILoggerDependency 
{ 
string GetCurrentDirectory(); 
string GetDirectoryByLoggerName(string loggerName); 
string DefaultLogger { get; } 	
}```
    
#### Реализация простой заглушки - `stub`

	ILoggerDependency loggerDependency =
   		 Mock.Of<ILoggerDependency>(d => d.GetCurrentDirectory() == "D:\\Temp")
	var currentDirectory = loggerDependency.GetCurrentDirectory();
 
	Assert.That(currentDirectory, Is.EqualTo("D:\\Temp"));
    
Простой объект с простым поведением. 

#### `stub` который зависит от входного параметра.
	Mock<ILoggerDependency> stub = new Mock<ILoggerDependency>();
 
	stub.Setup(ld => ld.GetDirectoryByLoggerName(It.IsAny<string>()))
    	.Returns<string>(name => "C:\\" + name);
 
	string loggerName = "SomeLogger";
	ILoggerDependency logger = stub.Object;
	string directory = logger.GetDirectoryByLoggerName(loggerName);
 
	Assert.That(directory, Is.EqualTo("C:\\" + loggerName));
    
это все еще `stub`, хотя и содержит некоторую логику.

#### Проверка поведения.
	public interface ILogWriter
	{
    	string GetLogger();
    	void SetLogger(string logger);
    	void Write(string message);
	}
	public class Logger
	{
    	private readonly ILogWriter _logWriter;
 
    	public Logger(ILogWriter logWriter)
    	{
        	_logWriter = logWriter;
    	}
 
    	public void WriteLine(string message)
    	{
        	_logWriter.Write(message);
    	}
	}
    //----------------------------------------------------
    var mock = new Mock<ILogWriter>();
	var logger = new Logger(mock.Object);
 
	logger.WriteLine("Hello, logger!");

	mock.Verify(lw => lw.Write(It.IsAny<string>()));
Таким образом мы проверяем, что метод Write вызвался с любым текстом на входе.

Можно также проверить что метод был вызван с конкретной строкой на входе, например так:
```mock.Verify(lw => lw.Write("Hello, logger!"));``` 

или проверить что метод был вызван ровно один раз

```mock.Verify(lw => lw.Write(It.IsAny<string>()),    Times.Once());```
#### Mock setup

Обычно применяется, когда необходимо провести какую-то групповую проверку нескольких условий. 

	var mock = new Mock<ILogWriter>();
	mock.Setup(lw => lw.Write(It.IsAny<string>()));
	mock.Setup(lw => lw.SetLogger(It.IsAny<string>()));
 
	var logger = new Logger(mock.Object);
	logger.WriteLine("Hello, logger!");
 
	mock.Verify();
    
Метод `Verify` не принимает никаких параметров, потому как все параметры уже установлены через `Setup`.

# CI

*Непрерывная интеграция* это прежде всего подход к разработке ПО. **Коротко** - это некоторая программа, которая наблюдает за состоянием кодовой базы и при необходимости производит сборку проекта, прогонку по тестам и стучит о результатах кому следует.




