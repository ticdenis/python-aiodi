# Python Dependency Injection library

aiodi is a Container for the Dependency Injection in Python.

## Installation

Use the package manager [pip](https://pypi.org/project/aiodi/) to install aiodi.

```bash
pip install aiodi
```

## Documentation

- Visit [aiodi docs](https://ticdenis.github.io/python-aiodi/).

## Usage

```python
from abc import ABC, abstractmethod
from logging import Logger, getLogger, NOTSET, StreamHandler, Formatter
from os import getenv

from aiodi import Container
from typing import Optional, Union

_CONTAINER: Optional[Container] = None


def get_simple_logger(
        name: Optional[str] = None,
        level: Union[str, int] = NOTSET,
        fmt: str = '[%(asctime)s] - %(name)s - %(levelname)s - %(message)s',
) -> Logger:
    logger = getLogger(name)
    logger.setLevel(level)
    handler = StreamHandler()
    handler.setLevel(level)
    formatter = Formatter(fmt)
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    return logger


class GreetTo(ABC):
    @abstractmethod
    def __call__(self, who: str) -> None:
        pass


class GreetToWithPrint(GreetTo):
    def __call__(self, who: str) -> None:
        print('Hello ' + who)


class GreetToWithLogger(GreetTo):
    _logger: Logger

    def __init__(self, logger: Logger) -> None:
        self._logger = logger

    def __call__(self, who: str) -> None:
        self._logger.info('Hello ' + who)


def container() -> Container:
    global _CONTAINER
    if _CONTAINER:
        return _CONTAINER
    di = Container({'env': {
        'name': getenv('APP_NAME', 'aiodi'),
        'log_level': getenv('APP_LEVEL', 'INFO'),
    }})
    di.resolve([
        (
            Logger,
            get_simple_logger,
            {
                'name': di.resolve_parameter(lambda di_: di_.get('env.name', typ=str)),
                'level': di.resolve_parameter(lambda di_: di_.get('env.log_level', typ=str)),
            },
        ),
        (GreetTo, GreetToWithLogger),  # -> (GreetTo, GreetToWithLogger, {})
        GreetToWithPrint,  # -> (GreetToWithPrint, GreetToWithPrint, {})
    ])
    di.set('who', 'World!')
    # ...
    _CONTAINER = di
    return di


def main() -> None:
    di = container()

    di.get(Logger).info('Just simple call get with the type')

    for greet_to in di.get(GreetTo, instance_of=True):
        greet_to(di.get('who'))


if __name__ == '__main__':
    main()

```

## Requirements

- Python >= 3.6

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://github.com/ticdenis/python-aiodi/blob/master/LICENSE)
