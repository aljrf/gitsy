Follow Nicole's tutorial at

https://notes.nicolevanderhoeven.com/How+to+publish+Obsidian+notes+with+Quartz+on+GitHub+Pages

- [x] install quartz

-used https://nodejs.org/en and then https://nodejs.org/en/download



npx quartz build --serve
browse at localhost:8080

create repository gitsy un github *without* a readme file

https://github.com/aljrf/gitsy.git

type 

```
git remote -v
```

output is

```
origin	https://github.com/jackyzha0/quartz.git (fetch)
origin	https://github.com/jackyzha0/quartz.git (push)
upstream	https://github.com/jackyzha0/quartz.git (fetch)
upstream	https://github.com/jackyzha0/quartz.git (push)
```
and replace the *origin* part with the repository:

```
git remote rm origin

```
```
git remote add origin https://github.com/aljrf/gitsy.git 
```

then the structure looks like 

```
aljrf@newdelly:~/gitsy$ git remote -v
origin	https://github.com/aljrf/gitsy.git (fetch)
origin	https://github.com/aljrf/gitsy.git (push)
upstream	https://github.com/jackyzha0/quartz.git (fetch)
upstream	https://github.com/jackyzha0/quartz.git (push)

```

then push everything in one go:
```
npx quartz sync --no-pull
```


