官方 github位置 --》https://github.com/mfreiholz/iF.SVNAdmin

配置文件：./data/

config.ini    #认证方式

userroleassignments.ini  #权限

iF.SVNAdmin 是用来直接管理 svn的认证文件 authz 和 passwd 简单好用
但是默认的passwd是基于http访问svn的  所以 passwd是根据htpassd加密的

我不需要使用http访问 只需要基于svn:// 的访问  passwd是明文的
所以 修改官方的iF.SVNAdmin 把加密改为明文
--------
标准的 改如下几部分   可实现不用加密passwd 
/var/www/html/svnadmin/include/ifcorelib/IF_HtPasswd.class.php

创建用户
 public function createUser( $username, $password, $crypt = true )
 改为
 public function createUser( $username, $password, $crypt = true )
  {
    if( self::userExists( $username ) )
    {
    	// The user already exists.
      $this->m_errno = 10;
      return false;
    }

  	// Should the password being crypted?
  	if( $crypt == false )  //modify 
  	{
  		$password = self::crypt_default( $password ); // Force MD5 as salt!
  	}

修改密码
  public function changePassword($username, $newpass, $crypt=true)
改为
  public function changePassword($username, $newpass, $crypt=false) //modify


认证用户   （取消加密，直接读文件明文认证）
---
  public function authenticate( $username, $password )
  {
  	// Find the user.
    foreach( $this->m_data as $usr=>&$pass )
    {
    	// Found user.
      if( $usr == $username )
      {
      	// Find out which encryption type is used.
      	// SHA
      	if (strpos($pass, "{SHA}") === 0)
--------
改为
  public function authenticate( $username, $password )
  {
  	// Find the user.
    foreach( $this->m_data as $usr=>&$pass )
    {
    	// Found user.
      if( $usr == $username ) //add
        if ($pass == $password) //add
          return true;	//add
        else	//add
          return false; //add
      	// Find out which encryption type is used.
      	// SHA
      	if (strpos($pass, "{SHA}") === 0)

写文件
  public function writeToFile( $filename = NULL )
改成如下
public function writeToFile( $filename = NULL )
  {
    if( $filename == NULL )
    {
      $filename = $this->m_userfile;
    }

    // Open file and write the array of users to it.
    $fh = fopen( $filename, "w" );
    flock( $fh, LOCK_EX );
    $svn_head = "[users]\n"; //add
    fwrite( $fh, $svn_head, strlen( $svn_head ) ); //add
    foreach( $this->m_data as $usr=>$pwd )
    {
      $line = $usr."=".$pwd."\n";
      fwrite( $fh, $line, strlen( $line ) );
    }
    flock( $fh, LOCK_UN );
    fclose( $fh );
    return true;
  }
