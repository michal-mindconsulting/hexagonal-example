# hexagonal-example

Repository interface

```php
<?php

declare(strict_types=1);

namespace Modules\Common\Repositories;

interface Repository
{
    public function find(int $id);

    public function findBy(string $column, $value);

    public function all();

    public function create(array $data);

    public function update(array $data);

    public function delete(string $column, $value);

    public function get();

    public function paginate(int $perPage);
}
```

Eloquent Base Repository

```php
<?php

declare(strict_types=1);

namespace Modules\Common\Repositories\Eloquent;

use Illuminate\Database\Eloquent\Builder;
use Modules\Common\Repositories\Eloquent\Criteria\Criteria;
use Modules\Common\Repositories\Repository as RepositoryInterface;

abstract class Repository implements RepositoryInterface
{
    /**
     * @var Builder
     */
    protected $query;

    public function __construct(){
        $this->query = $this->newQueryBuilder();
    }

    public abstract function model(): string;

    public function newQueryBuilder(): Builder{
        return $this->model()::query();
    }

    public function find(int $id){
        return $this->query->find($id);
    }

    public function findBy(string $column, $value){
        return $this->query->where($column, $value)->first();
    }

    public function all(){
        return $this->newQueryBuilder()->get();
    }

    public function create(array $data){
        return $this->newQueryBuilder()->create($data);
    }

    public function update(array $data){
        return $this->newQueryBuilder()->update($data);
    }

    public function delete(string $column, $value){
        return $this->newQueryBuilder()->where($column, $value)->delete();
    }

    public function apply(Criteria $criteria){
        $criteria->apply($this->query);
    }

    public function get(){
        return $this->query->get();
    }

    public function paginate(int $perPage){
        return $this->query->paginate($perPage);
    }
```

Example UserRepositoryInterface

```php
<?php

declare(strict_types=1);

namespace Modules\Customer\Repository;

use Modules\Common\Repositories\Repository;
use Modules\Customer\Dto\UserFilterDto;

interface UserRepositoryInterface extends Repository
{
    public function filter(UserFilterDto $dto): UserRepositoryInterface;
}

```

Example UserRepository eloquent implementation

```php
<?php

declare(strict_types=1);

namespace Modules\Customer\Repository\Eloquent;

use App\Models\User;
use Modules\Common\Repositories\Eloquent\Repository;
use Modules\Customer\Dto\UserFilterDto;
use Modules\Customer\Repository\UserRepositoryInterface;

class UserRepository extends Repository implements UserRepositoryInterface
{
    public function model(): string
    {
        return User::class;
    }

    public function filter(UserFilterDto $dto): UserRepositoryInterface{
        if( !is_null($dto->getName()) ){
            $this->query->where('users.name', $dto->getName());
        }

        if( !is_null($dto->getEmail()) ){
            $this->query->where('users.email', $dto->getEmail());
        }

        return $this;
    }
}

```
