### 6 추상화 예제
예제
- 클라우드 파일 통합 관리 기능 개발
- 대상 클라우드 : 드롭박스, 박스
- 주요 기능 : 각 클라우드의 파일 목록 조회, 다운로드, 업로드, 삭제, 검색

추상화 하지 않은 구현 : 파일 목록 조회
```java
public enum CloudId {
  DROPBOX,
  BOX;
}

public class FileInfo {
  private CloudId cloudId;
  private String fileId;
  private String name;
  private long length;
  ...
}

public class CloudFileManager {
  public List<ListInfo> getFileInfos(CloudId cloudId) {
    if (cloudId == CloudId.DROPBOX) {
      DropboxClient db = ...;
      List<DbFile> dbFiles = db.getFiles();
      List<FileInfo> result = new ArrayList<>();
      for (DbFile dbFile : dbFiles) {
        FileInfo fi = new FileInfo();
        fi.setCloudId(CloudId.DROPBOX);
        fi.setFileId(fi.getFileId());
        ...
        result.add(fi);
      }
      return result;
    } else if (cloudId == CloudId.BOX) {
      BoxService boxSvc = ...;
      ...
    }
  }
}
```

추상화 하지 않은 구현 : 파일 다운로드
```java
public void download(FileInfo file, File localTarget) {
  if (file.getCloudId() == CloudId.DROPBOX) {
    DropboxClient dc = ...;
    FileOutputStream out = new FileOutputStream(localTarget);
    db.copy(file.getFileId(), out);
    out.close();
  } else if (file.getCloudId() == CloudId.BOX) {
    BoxService boxSvc = ...;
    InputStream is = boxSvc.getInputStream(fileId.getId());
    FileOutputStream out = new FileOutputStream(localTarget);
    CopyUtil.copy(is, out);
  }
}
```

추상화 하지 않은 구현 : 기타 기능 구현
```java
public FileInfo upload(File file, CloudId cid) {
  if (cid == CloudId.DROPBOX) {
    ...
  } else if (cid == CloudId.BOX) {
    ...
  }
}

public void delete(String fileId, CloudId cid) {
  if (cid == CloudId.DROPBOX) {
    ...
  } else if (cid == CloudId.BOX) {
    ...
  }
}

public List<FileInfo> search(String query, CloudId cid) {
  if (cid == CloudId.DROPBOX) {
    ...
  } else if (cid == CloudId.BOX) {
    ...
  }
}
```

추상화 하지 않은 구현 : 요구사항 발생
- 추가 구현
  - 클라우드 추가 : S클라우드, N클라우드, D클라우드
  - 기능 추가 : 클라우드 간 파일 복사

추상화하지 않은 구현 : 클라우드 추가

```java
public List<ListInfo> getFileInfos(CloudId cloudId) {
  if (cloudId == CloudId.DROPBOX) {
    ...
  } else if (cloudId == CloudId.BOX) {
    ...
  } else if (cloudId == CloudId.SCLOUD) {
    ...
  } else if (cloudId == CloudId.NCLOUD) {
    ...
  } else if (cloudId == CloudId.DCLOUD) {
    ...
  }
}
```
- download(), upload(), delete(), search() 등에도 유사한 else-if 블록 추가됨

추상화하지 않은 구현 : 클라우드 간 복사
```java
public FileInfo copy(FileInfo fileInfo, CloudId to) {
  CloudId from = fileInfo.getCloudId();
  if (cloudId == CloudId.DROPBOX) {
    DropBoxClient dbClient = ...;
    if (cloudId == CloudId.BOX) {
      dbClient.copyFromUrl("http://www.box.com/files/" + fileInfo.getFileId());
    } else if (cloudId == CloudId.SCLOUD) {
      ScloudClient sClient = ...;
      InputStream is = sClient.getInputStream(fileInfo.getFileId());
      dbClient.copyFromInputStream(is, fileInfo.getName());
    } else if (cloudId == CloudId.NCLOUD) {
      NCloudClient nClient = ...;
      File temp = File.createTemp();
      nClient.save(fileInfo.getFileId(), temp);
      InputStream is = new FileInputStream(temp);
      dbClient.copyFromInputStream(is, fileInfo.getName());
    } else if (cloudId == CloudId.DCLOUD) {
      dbClient.copyFromUrl("http://dcloud.com/getFile?fileId=" + fileInfo.getFileId());
    }
  } else if(...) {
      ...
  } ...  
}
```
- 수많은 if-else 구문에 로직 구현됨
- 코드 변경 비용 증가!

개발 시간 증가
- 코드 구조 복잡해짐
  - 새로운 클라우드 추가시 모든 메서드에 새로운 if 블록 추가
    - 중첩 if-else는 복잡도 배로 증가
    - if-else 많을 수록 진척 느려짐
- 관련 코드가 여러 곳에 분산됨
  - 한 클라우드 처리와 관련된 코드가 여러 메서드에 분산됨
- 결론적으로, 코드 가독성, 분석 속도 저하
  - 코드 추가에 따른 시간 증가
  - 휴먼 에러 가능성 증가, 디버깅 시간 증가

추상화하여 재설계
- CloudFileSystemFactory
  - getFileSystem
- (Interface) CloudFileSystem
  - getFiles, search, getFile, addFile
- (Interface) CloudFile
  - getId, getName, getLength, hasUrl, getUrl, getInputStream, write, delete

추상화 구현 : DropBoxFileSystem
```java
public class DropBoxFileSystem implements CloudFileSystem {
  private DropBoxClient dbClient = new DropBoxClient(...);

  @Override
  public List<CloudFile> getFiles() {
    List<DbFile> dbFiles = dbClient.getFiles();
    List<CloudFile> results = new ArrayList<>(dbFiles.size());
    for (DbFile file : dbFiles) {
      DropBoxCloudFile cf = new DropBoxCloudFile(file, dbClient);
      results.add(Cf);
    }
    return result;
  }
}
```

추상화 구현 : DropBoxCloudFile
```java
public class DropBoxCloudFile implements CloudFile {
  private DbFile dbFile;
  private DropBoxClient dbClient;

  public DropBoxCloudFile(DbFile dbFile, dbClient) {
    this.dbFile = dbFile;
    this.dbClient = dbClient;
  }

  public String getId() {
    return dbFile.getId();
  }
  public boolean hasUrl() {
    return true;
  }
  public String getUrl() {
    return dbFile.getFileUrl();
  }
  public String getName() {
    return dbFile.getFileName();
  }
  public InputStream getInputStream() {
    return dbClient.createStreamOfFile(dbFile);
  }
  public write(OutputStream out) {
    ...
  }
  public void delete() {
    dbClient.deleteFile(dbFile.getId());
  }
  ...
}
```

추상화 구현 : 파일 목록, 다운로드 기능
```java
public List<CloudFile> getFileInfos(CloudId cloudId) {
  CloudFileSystem fileSystem = CloudFileSystemFactory.getFileSystem(cloudId);
  return fileSytem.getFiles();
}

public void download(CloudFile file, File localTarget) {
  file.write(new FileOutputStream(localTarget));
}
```

추상화 구현 : BOX 클라우드 지원 기능 추가
- DropBox 와 동일한 형태로 BoxFileSystem, BoxCloudFile 클래스 구현 추가
- 파일 목록, 다운로드 기능 구현 코드 변경 필요 없음

추상화 구현 : 파일 복사 기능 추가

```java
public void copy(CloudFile file, CloudId target) {
  CloudFileSystem fileSystem = CloudFileSystemFactory.getFileSystem(target);
  fileSytem.copyFrom(file);
}
```

```java
// DropBoxFileSystem
private DropBoxClient dbClient = new DropBoxClient(...);
public void copyFrom(ClouFile file) {
  if (file.hasUrl()) {
    dbClient.copyFromUrl(file.getUrl());
  } else {
    dbClient.copyFromInputStream(file.getInputStream(), file.getName());
  }
}
```

```java
// NcloudFileSystem
private NcloudClient nClient = new NcloudClient(...);
public void copyFrom(ClouFile file) {
  File tempFile = File.createTemp();
  file.write(new FileOutputStream(tempFile));
  nClient.upload(tempFile, file.getName());
}
```

추상화 결과 : 추상화한 타입으로만 핵심 기능 구현

```java
public class CloudFileManager {
  public List<CloudFile> getFileInfos(CloudId cloudId) {
    CloudFileSystem fileSystem = CloudFileSystemFactory.getFileSystem(cloudId);
    return fileSytem.getFiles();
  }
  public void download(CloudFile file, File localTarget) {
    file.write(new FileOutputStream(localTarget));
  }
  public void copy(CloudFile file, CloudId target) {
    CloudFileSystem fileSystem = CloudFileSystemFactory.getFileSystem(target);
    fileSytem.copyFrom(file);
  }
}
```
- CloudFileManager 코드 변경없이 새로운 클라우드 지원 추가 가능
- 특정 Cloud 관련 코드를 모아서 관리


OCP
- 개방 폐쇄의 원칙 (Open-Closed Principle)
- 확장에 열려 있음 (Open for Extention) - 새로운 기능 추가가 쉬워야한다
- 변경에 닫혀 있음 (Closed for Modification) - 기능을 사용하는 코드는 수정하지 않아야한다
