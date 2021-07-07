---
title: "Upload and Download File by SFTP"
date: 2021-07-06T08:06:25+06:00
description: "write Java code that uploads a file from local computer to a remote SFTP server, or download a file from remote server to local directory"
menu:
  sidebar:
    name: Upload and Download a File by SFTP
    identifier: content-JAVA-FTP-directory
    parent : File Transfer-directory
    weight: 10
hero: images/upload.jpg
---
When data is sent to another party, it's potentially exposed to outside entities on the Internet, FTP and SFTP are two common file transfer options, Learning more about them will make a proper choice for sharing your company’s data.

## FTP v.s SFTP
FTP is file transfer protocol. It's a simple way of transferring data, making data readable for anyone who intercepts it. While this is fine if you’re just sending unimportant files.

SFTP is Secure File Transfer Protocol, it means what it says, safer than FTP.
SFTP uses SSH (or secure shell) encryption to protect data from opening to potential breaches and compromises to the company.

## Example
We can upload or download any files, including a text, a pictrure or audios by FTP and SFTP.
Here's the demonstration of how to do code it. First let's put JSch dependency in pom.xml so we can integrate its functionality into Java programs.
```
<dependency>
	<groupId>com.jcraft</groupId>
	<artifactId>jsch</artifactId>
	<version>0.1.55</version>
</dependency>
```
## Download File
import the following classes
```
import com.jcraft.jsch.Channel;
import com.jcraft.jsch.ChannelSftp;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.JSchException;
import com.jcraft.jsch.Session;
import com.jcraft.jsch.SftpException;
```
Instantiates the Session object with given username, host and port. 
This Session represents a connection to a SSH server. One session can contain multiple Channels of various types, created with openChannel(), opened with connect() and closed with disconnect().

```
        Session session = null;
        Channel channel = null;

        //Your Custom connection infos, usually encapsulate these infos inside a bean
        String UserId = "myId";
        String host = "10.10.10.10";
        int port = 22;
        String password = "myPwd";
        String remotePath = "/home/myRemotePath";
        String remoteFileName = "myFile";
        String localDownloadFolder = "C:/downloads";

        try {
            JSch jsch = new JSch();
            session = jsch.getSession(UserId, host, port);

            // connect without verifying SSH host keys
            session.setConfig("StrictHostKeyChecking", "no");
            session.setPassword(password);
            session.connect();

            channel = session.openChannel("sftp");
            channel.connect();
            ChannelSftp channelSftp = (ChannelSftp) channel;

            channelSftp.cd(remotePath);

            int download_index = 0;
                try {
                     channelSftp.get(remoteFileName, localDownloadFolder);
                    } catch (SftpException sftpException) {
                     LOGGER.error("Download file fail." , sftpException);
                   } 
        } catch (Exception exception) {
            LOGGER.error("------ Sftp download file fail." ,exception);
            throw exception;
        } finally {
            // close SFTP
            channelSftp.exit();
            SftpUtils.clodeSftpConnection(channel,session);
        }
    
```
## Upload File
For uploading file, 2 more imports are needed
```
import java.io.File;
import java.io.FileInputStream;
```
The code cover up most of the exceptions might encounter while uploading with try catch blocks and loggers.
```
        Session session = null;
        Channel channel = null;
        //Your Custom connection infos, usually encapsulate these infos inside a bean
        String UserId = "myId";
        String host = "10.10.10.10";
        int port = 22;
        String password = "myPwd";
        String remotePath = "/home/myRemotePath";
        String localFilePath = "C:/downloads/file.txt";

        File uploadFile = new File(localFilePath);
        String uploadFileName = uploadFile.getName(); //file.txt

        try (FileInputStream fis = new FileInputStream(uploadFile)) {
            JSch jsch = new JSch();
            session = jsch.getSession(UserId, host, port);

            // connect without verifying SSH host keys
            session.setConfig("StrictHostKeyChecking", "no");
            session.setPassword(password);
            session.connect();

            channel = session.openChannel("sftp");
            channel.connect();
            ChannelSftp channelSftp = (ChannelSftp) channel;

            try {
                channelSftp.mkdir(remotePath);

            } catch (SftpException sftpException) {
                if(!StringUtils.equals("Failure",sftpException.getMessage())){
                    LOGGER.error("fail to create SFTP direcrory , reason is {} ",sftpException);
                }
            }
            // combine remoteFilePath and fileName
            String remoteFileEntirePath = remotePath + "/" + uploadFileName;
            LOGGER.info("*** Entire path of remote file is = {}", remoteFileEntirePath);
            channelSftp.put(fis, remoteFileEntirePath, ChannelSftp.OVERWRITE);
            LOGGER.info("======>Upload Success!");
        } catch (Exception exception) {
            LOGGER.error("upload file to remote ftp fail.", exception);
        } finally {
            //close SFTP connection
            try {
                if(channel!=null && channel.isConnected()){
                    channel.disconnect();
                    LOGGER.debug("Disconnect channel => {}",!channel.isConnected() );
                    LOGGER.debug("Close channel => {}",channel.isClosed());
                }
                if(session!=null && session.isConnected()){
                    session.disconnect();
                    LOGGER.debug("Disconnect session => {}",!session.isConnected());
                }
            } catch (Exception exception) {
                LOGGER.error("------ Closed Sftp fail.", exception);
            }
        }

    }
```
