# Using datamode-code-generator as a Module

datamodel-code-generator is a CLI tool, but it can also be used as a module.

You can call this code-generator in your python code.

## How to use it as module

You can generate models with `datamodel_code_generator.generate` using parameters that match the [arguments provided to the CLI tool](./index.md#all-command-options). The generated files can be written to and read from the `Path` object supplied to *output*.

In the below example, we use a file in a `TemporaryDirectory` to store our output.

### Installation
```sh
pip install "datamodel-code-generator[http]"
```

### Example
```python
from pathlib import Path
from tempfile import TemporaryDirectory
from datamodel_code_generator import InputFileType, generate
from datamodel_code_generator import DataModelType
    
json_schema: str = """{
    "type": "object",
    "properties": {
        "number": {"type": "number"},
        "street_name": {"type": "string"},
        "street_type": {"type": "string",
                        "enum": ["Street", "Avenue", "Boulevard"]
                        }
    }
}"""

with TemporaryDirectory() as temporary_directory_name:
    temporary_directory = Path(temporary_directory_name)
    output = Path(temporary_directory / 'model.py')
    generate(
        json_schema,
        input_file_type=InputFileType.JsonSchema,
        input_filename="example.json",
        output=output,
        # set up the output model types
        output_model_type=DataModelType.PydanticV2BaseModel,
    )
    model: str = output.read_text()
print(model)
```

The result of `print(model)`:
```python
# generated by datamodel-codegen:
#   filename:  example.json
#   timestamp: 2020-12-21T08:01:06+00:00

from __future__ import annotations

from enum import Enum
from typing import Optional

from pydantic import BaseModel


class StreetType(Enum):
    Street = 'Street'
    Avenue = 'Avenue'
    Boulevard = 'Boulevard'


class Model(BaseModel):
    number: Optional[float] = None
    street_name: Optional[str] = None
    street_type: Optional[StreetType] = None
```


Also, you can call parser directly

```python
from datamodel_code_generator import DataModelType, PythonVersion
from datamodel_code_generator.model import get_data_model_types
from datamodel_code_generator.parser.jsonschema import JsonSchemaParser

json_schema: str = """{
    "type": "object",
    "properties": {
        "number": {"type": "number"},
        "street_name": {"type": "string"},
        "street_type": {"type": "string",
                        "enum": ["Street", "Avenue", "Boulevard"]
                        }
    }
}"""


data_model_types = get_data_model_types(
    DataModelType.PydanticV2BaseModel,
    target_python_version=PythonVersion.PY_311
)
parser = JsonSchemaParser(
   json_schema,
   data_model_type=data_model_types.data_model,
   data_model_root_type=data_model_types.root_model,
   data_model_field_type=data_model_types.field_model,
   data_type_manager_type=data_model_types.data_type_manager,
   dump_resolve_reference_action=data_model_types.dump_resolve_reference_action,
                       )
result = parser.parse()
print(result)
 ```

The result of `print(model)`:
```python
from __future__ import annotations

from enum import Enum
from typing import Optional

from pydantic import BaseModel


class StreetType(Enum):
    Street = 'Street'
    Avenue = 'Avenue'
    Boulevard = 'Boulevard'


class Model(BaseModel):
    number: Optional[float] = None
    street_name: Optional[str] = None
    street_type: Optional[StreetType] = None

```
## Why doesn't `datamodel_code_generator.generate` return a string? 

The above example schema only generates a single python module, but a single schema may generate multiple modules. There is no way to represent these modules as a single string, so the `generate` method returns `None`.

Note that the *output* parameter can take any `Path` object, which includes both file and directory paths. If a file name is provided and multiple modules are generated, `generate` will raise a `datamodel_code_gen.Error` exception.

If multiple modules are generated, you will need to walk through the supplied *output* directory to find all of them.
