# Transformação de Dados para Iniciantes



## TOMO I 

Para explicar de forma simples e lúdica irei utilizar como massa de dados o retorno de uma 
requisição na [API do Redtube](http://api.redtube.com/) onde teremos, como exemplo, esse objeto:

```js
{
    "video": {
        "duration": "14:08",
        "views": "43110",
        "video_id": "1103943",
        "rating": "3.86",
        "ratings": "236",
        "title": "Loira dando o bucet&atilde;o e o c&uacute; de quatro",
        "url": "http://www.redtube.com/1103943",
        "embed_url": "https://embed.redtube.com/?id=1103943",
        "default_thumb": "https://thumbs-cdn.redtube.com/m=e0YH8f/_thumbs/0001103/1103943/1103943_004o.jpg",
        "thumb": "https://thumbs-cdn.redtube.com/m=e0YH8f/_thumbs/0001103/1103943/1103943_004o.jpg",
        "publish_date": "2017-02-11 13:30:01",
        "tags": [
          {
            "tag_name": "Amateur"
          },
          {
            "tag_name": "Anal Sex"
          },
          {
            "tag_name": "Big Ass"
          },
          {
            "tag_name": "Brazilian"
          },
          {
            "tag_name": "Couple"
          },
          {
            "tag_name": "Homemade"
          },
          {
            "tag_name": "Latin"
          },
          {
            "tag_name": "POV"
          },
          {
            "tag_name": "Shaved"
          },
          {
            "tag_name": "Vaginal Sex"
          }
        ]
      }
  }
  
```



Ao final desse conteúdo vc deverá estar apto a resolver o seguinte problema:


>Liste quais vídeos possuem duração entre 2 e 5 minutos, que possui pelo menos 100 `views` com 
>uma média de avaliação maior que 2 e sua publicação foi na última semana.

Para resolver essa pendenga precisaremos basicamente de 3 funções:

- map
- filter
- reduce


### filter

- filtrar
  + por `duration`
    * menor que
    * maior que
    * entre dois valores


> Irei ensinar como filtrar por `duration` e o resto dos outros filtros vc terá que fazer!

Então vamos começar pelo `menor que`, ou seja, pela duração máxima:

```js

const filterByMaxDuration = ( videos, max ) => {

  const byMaxDuration = ( obj ) => obj.video.duration.split(':')[0] <= max

  return  videos.filter( byMaxDuration )

}

console.log('filterByMaxDuration', filterByMaxDuration(videos, 2) )
```

Obviamente vc percebeu que ele testa apenas os minutos, para transformar todas as durações em 
segundos teremos que usar o `map` e isso é um pouco mais para frente.

Na função `filterByMaxDuration` pegamos o valor da duração com `obj.video.duration` para depois 
executarmos o `.split(':')` para separar os minutos dos segundos e com `[0]` pegamos apenas o 
valor dos minutos. 

Logo ela só irá retornar videos com duração, dos minutos, menor ou igual ao valo passado para 
`filterByMaxDuration`.

Agora ficou fácil fazer as outras funções:

```js
const filterByMaxDuration = ( videos, max ) => {

  const byMaxDuration = ( obj ) => obj.video.duration.split(':')[0] <= max

  return  videos.filter( byMaxDuration )

}

const filterByMinDuration = ( videos, min ) => {

  const byMinDuration = ( obj ) => obj.video.duration.split(':')[0] >= min

  return  videos.filter( byMinDuration )

}

const filterByRangeDuration = ( videos, min, max ) => {

  const byRangeDuration = ( obj ) =>  obj.video.duration.split(':')[0] >= min
                                                      && obj.video.duration.split(':')[0] <= max

  return  videos.filter( byRangeDuration )

}

```


Para melhorar isso podemos criar uma função de mediação para termos apenas 1 função de filtragem 
pela duração, por exemplo:

```js

const filterDurationBy = ( videos, type, initial=0, end=0) => {

  switch ( type ) {
    case 'range':
      return filterByRangeDuration( videos, initial, end )
      break;
    case 'max':
      return filterByMaxDuration( videos, initial )
      break;
    case 'min':
      return filterByMinDuration( videos, initial ) 
      break;
    default:
      return console.log('Essa opção não existe!')
      break;
  }
}
console.log('\n\n\n filterDurationBy', filterDurationBy(videos, 'min', 40) )
console.log('\n\n\n filterDurationBy', filterDurationBy(videos, 'max', 2) )
console.log('\n\n\n filterDurationBy', filterDurationBy(videos, 'range', 40, 50) )

```

Deixando nosso módulo final e refatorado assim:

```js

const filterDurationBy = ( videos, type, initial=0, end=0) => {

  const byMaxDuration = ( obj ) => obj.video.duration <= initial
  const byMinDuration = ( obj ) => obj.video.duration >= initial
  const byRangeDuration = ( obj ) =>  obj.video.duration >= initial
                                                        && obj.video.duration <= end


  const filterDurationByMax = ( videos, max ) => videos.filter( byMaxDuration )
  const filterDurationByMin = ( videos, min ) => videos.filter( byMinDuration )
  const filterDurationByRange = ( videos, min, max ) => videos.filter( byRangeDuration )

  switch ( type ) {
    case 'range':
      return filterDurationByRange( videos, initial, end )
      break;
    case 'max':
      return filterDurationByMax( videos, initial )
      break;
    case 'min':
      return filterDurationByMin( videos, initial ) 
      break;
    default:
      return console.log('Essa opção não existe!')
      break;
  }
}

```

Percebeu que eu separei as funções de filtragem para ficar mais legível e de fácil manutenção?


**Entretanto ainda podemos melhorar atomizando cada função de `filter` assim:**


```js

const filterDurationBy = ( initial=0 ) => ( obj ) =>  obj.video.duration <= initial

module.exports = filterDurationBy

```

```js

const filterDurationBy = ( initial=0 ) => ( obj ) =>  obj.video.duration >= initial

module.exports = filterDurationBy

```

```js

const filterDurationBy = ( initial=0, end=0 ) => ( obj ) =>  obj.video.duration >= initial
                                                                                      && obj.video.duration <= end

module.exports = filterDurationBy

```


**Para que possamos utiliza-las da seguinte forma:**

```js
const filterByDuration = ( videos, type, initial=0, end=0) => {

  const byMaxDuration = require('./actions/filterDurationByMax')(initial)
  const byMinDuration = require('./actions/filterDurationByMin')(initial)
  const byRangeDuration = require('./actions/filterDurationByRange')(initial, end)


  const filterByMaxDuration = ( videos, max ) => videos.filter( byMaxDuration )
  const filterByMinDuration = ( videos, min ) => videos.filter( byMinDuration )
  const filterByRangeDuration = ( videos, min, max ) => videos.filter( byRangeDuration )

  switch ( type ) {
    case 'range':
      return filterByRangeDuration( videos, initial, end )
      break;
    case 'max':
      return filterByMaxDuration( videos, initial )
      break;
    case 'min':
      return filterByMinDuration( videos, initial ) 
      break;
    default:
      return console.log('Essa opção não existe!')
      break;
  }
}

module.exports = filterByDuration

```


> **Agora é a sua vez de fazer as outras funçōes!**


#### Exercícios

```

1) Criar as funções abaixo para filtrar, de forma atômica e modular.

2) Criar 1 exemplo para cada filter

```


1)
- por `views`
  + menor que
  + maior que
  + entre dois valores
- por `ratings`
  + menor que
  + maior que
  + entre dois valores
- por `publish_date`
  + menor que
  + maior que
  + entre dois valores

### map

Será com o `map` que conseguiremos modificar os valores já existentes, por exemplo:

- transformar HH:MM:SS em **segundo**

Sabendo que os valores são separados por `:`,  iremos criar uma função que os separa, por exemplo:

```js

const breakDuration = ( obj ) => obj.video.duration.split(':')

```

Com o retorno dessa função já saberemos se a duração possui apenas segundos ou minutos ou horas, 
apenas verificando o tamanho do *Array* retornado. Agora ficou fácil fazermos essa função, olhe:

```js

const transformToSeconds = ( obj ) => {
  const time = obj.video.duration.split(':')
  let seconds = 0

  if ( time.length === 3 ) {
      seconds += parseInt( time[0] * 3600 )
      seconds += parseInt( time[1] * 60 )
      seconds += parseInt( time[2] )
    }
  if ( time.length === 2 ) {
      seconds += parseInt( time[0] * 60 )
      seconds += parseInt( time[1] )
    }
    obj.video.duration = seconds
    return obj
}


const transformDuration = ( obj ) => obj.map( transformToSeconds )

console.log('\n\n\n transformDuration', transformDuration(videos) )

```


Claro que podemos melhora-la! Perceba aqueles `ifs`, o segundo `if` não é necessário pois mesmo 
que a duração possua apenas segundos ela virá **SEMPRE** com 0 nos minutos, logo podemos deixar assim:

```js

const transformToSeconds = ( obj ) => {
  const time = obj.video.duration.split(':')
  let seconds = 0

  if ( time.length === 3 ) 
    seconds += parseInt( time[0] * 3600 )
    
  seconds += parseInt( time[0] * 60 )
  seconds += parseInt( time[1] )

  obj.video.duration = seconds
  return obj
}

module.exports = transformToSeconds

```


Vamos exportar essa função que usaremos direto no `map`, em conjunto do nosso `filter`, ficando assim:

```js

const {videos} = require('./../jsons/redtube.brazil.json')

const filterDuration = require('./filter.duration.seconds')
const transformDuration = require('./map.duration')

const myVideos = ( videos, type, min, max ) => 
  filterDuration( videos.map( transformDuration ), type, min, max )

const type = 'range'
const min = 30
const max = 60

console.log('myVideos', myVideos( videos, type, min, max ))

```


> **Percebeu que estou usando outro módulo de `filter`?**


Salvei como o nosso antigo, retirando apenas o `split` pois nesse módulo ele irá trabalhar diretamente
com segundos, por isso existiu essa necessidade de modificação.

```js

const byMaxDuration = ( obj ) => obj.video.duration <= initial
const byMinDuration = ( obj ) => obj.video.duration >= initial
const byRangeDuration = ( obj ) =>  obj.video.duration >= initial
                                                        && obj.video.duration <= end

```

Agora veja como iremos utilizar essas funções:

```js

const {videos} = require('./../jsons/redtube.brazil.json')

const filterDurationByRange = require('./actions/filterDurationByRange.js')(30, 60)
const transformDuration = require('./map.duration')

const myVideos = ( videos ) => videos
                                                    .map( transformDuration )
                                                    .filter( filterDurationByRange )


console.log('myVideos', myVideos( videos ))

```


> **Percebeu algo de diferente?**


Então analise esse código: `require('./actions/filterDurationByRange.js')(30, 60)`

Note que modularizei **APENAS** a função `filterDurationByRange` e para conseguir pegar o `min` e `max` 
precisamos criar um módulo que receba esses valores e depois retorne a função que usaremos.


```js

const filterDurationBy = ( initial=0, end=0 ) => ( obj ) =>  obj.video.duration >= initial
                                                                                      && obj.video.duration <= end


module.exports = filterDurationBy

```

Com isso a função que usaremos no `filter` terá acesso aos valores de `min` e `max` pois vc criou uma: [CLOSURE]()
