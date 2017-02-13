Workaround for Git Large files detected
===

GitHubに誤って100Mを超えるようなファイルをpushしてしまうと以下のようなエラーが出てpushに失敗する.

```
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
```

このときは以下のように対処する.Large fileは`large-file.jar`とする.

```bash
$ git rm --cached path/2/large-file.jar

$ git cm --amend -m "delete too large files"

$ git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/2/large-file.jar' \
    --prune-empty --tag-name-filter cat -- --all

# push again
$ git push origin some-branch-name
```

pushしたい場合はlfsを検討する.
