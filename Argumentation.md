Ответ основан на результатах веб-поиска. Вот краткое изложение преимуществ управления памяти в языках программирования C++, Java, Rust, Python и Go:

- C++ дает полный контроль над управлением памяти с помощью ручного выделения и освобождения памяти, но это может приводить к ошибкам, таким как утечки памяти или повреждение памяти.
- Java использует автоматическое управление памяти с помощью сборщика мусора, который периодически очищает неиспользуемую память, но это может снижать производительность и потреблять больше памяти.
- Rust предлагает новый подход к управлению памяти, основанный на концепции владения и заимствования. Rust проверяет правильность использования памяти на этапе компиляции и гарантирует безопасность и скорость, равную C и C++, без необходимости ручного управления или сборки мусора.
- Python также использует автоматическое управление памяти с помощью сборщика мусора, но он менее эффективен, чем в Java, и может приводить к фрагментации памяти. Python также медленнее, чем другие языки, из-за его динамической типизации и интерпретации.
- Go похож на Java в том, что он также использует автоматическое управление памяти с помощью сборщика мусора, но он оптимизирован для параллельных и конкурентных программ. Go быстрее, чем Python, но медленнее, чем Rust или C++.

Источник: беседа с Bing, 24.04.2023(1) Memory Management - Comprehensive Rust 🦀. https://google.github.io/comprehensive-rust/memory-management.html Получен доступ 24.04.2023.
(2) Rust vs. Go: How to choose | InfoWorld - Channel Asia. https://www.infoworld.com/article/3436960/rust-vs-go-how-to-choose.html Получен доступ 24.04.2023.
(3) Rust vs. Python: Could Rust replace Python? - LogRocket Blog. https://blog.logrocket.com/rust-vs-python-could-rust-replace-python/ Получен доступ 24.04.2023.


Почему я захотел создать свой язык? Потому что во время написания очередной программы на каком-нибудь языке программирования мне часто приходила мысль: "Почему такую простую вещь нужно делать так сложно? Зачем писать по нескольку ключевых слов, когда это можно сделать одним? Почему одну и ту же вещь можно сделать несоклькими способами и какие-то из них в сообществе считаются не предпочтительными?". На все эти негодования я записывал свой способ, как бы я реализовал ту или иную вещь. Можно подумать, что такие идеи будут вряд ли удобны программистам, поскольку это моё субъективное виденье. И вообще, зачем переписывать то, что придумали большие дядьки? Но, когда я стал больше углубляться в эту тему, изучать другие языки и их конструкции, я заметил, что в относительно новых языках, таких как Rust и Dart реализованы в точнсти такие же идеи, о которых подумал я. Поэтому, явно можно сказать, что недоволен некоторыми инструментами и приходят в голову такие идеи не одному мне.

Пояснить, почему нет: макросов, ;, кастомных операторов, слов in and or not, скобки в if необязательны

Всего есть 3 вида передачи параметра: по значению, по ссылке, перемещение (move). Самый часто используемый - по ссылке. Но плюсы по умолчанию используют передачу по значению, что пораждает кучу ненужного синтаксиса. Я же решил по умолчанию сделать передачу по ссылкам, а чтобы передать по ссылке или переместить, достаточно поставить соответствующий символ перед переменной.

Почему я перешёл от модели памяти схожей с Rust к модели, которая используется сейчас?
Во-первых, неизменяемые переменные по умолчанию - удобно для компилятора, но не для программиста. Писать везде mut - совсем не коротко, тем более, язык Aqua должен быть языком где нет ничего лишнего и если можно что-то сделать короче - то так и должно быть сделано. По этой причине я даже отказался от ключевого слова для объявления функций. К тому же, нужно держать в голове тонкую разницу между mut и const.