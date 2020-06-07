# SpringBoot实现文件上传下载

````java
@Controller
public class FileUpDownController {

	//单文件上传
	@PostMapping("/fileUpLoad")
	@ResponseBody
	public String fileUpLoad(@RequestParam("file")MultipartFile file) {
		    String msg="";
			String originalFilename = file.getOriginalFilename();
			String path = this.getClass().getClassLoader().getResource("").getPath();
			try {
				File fileCopy = new File(path+"\\file\\"+originalFilename);
				//if 该文件的父目录不存在，即该文件也不存在；则手动创建该文件父目录、该文件
				if (!fileCopy.getParentFile().exists()) {
					fileCopy.getParentFile().mkdirs();
					fileCopy.createNewFile();
				}
				file.transferTo(fileCopy);
				msg="上传成功";
			} catch (IllegalStateException | IOException e) {
				e.printStackTrace();
				msg="上传失败";
			}
		return msg;
		}
	
	//多文件上传
	@PostMapping("/fileUpLoads")
	@ResponseBody
	public String fileUpLoads(@RequestParam("file")MultipartFile[] files) {
		    String msg="";
		    try {
		    	if (files.length>0) {
			    	for (MultipartFile file : files) {
				    	String originalFilename = file.getOriginalFilename();
						String path = this.getClass().getClassLoader().getResource("").getPath();
						File fileCopy = new File(path+"\\file\\"+originalFilename);
						//if 该文件的父目录不存在，即该文件也不存在；则手动创建该文件父目录、该文件
						if (!fileCopy.getParentFile().exists()) {
							fileCopy.getParentFile().mkdirs();
							fileCopy.createNewFile();
						}
						file.transferTo(fileCopy);
					}
			    	msg="上传成功";
				}
			} catch (Exception e) {
				e.printStackTrace();
				msg="上传失败";
				//此处根据需求是否需要回滚（将部分成功的文件删除，脏文件）
			}
		return msg;
		}
	
	//单文件下载
	@GetMapping("/fileDownLoad")
	public ResponseEntity<byte[]> fileDownLoad(@RequestParam("filename") String filename){
		String filePath = this.getClass().getClassLoader().getResource("file/"+filename).getPath();
		System.out.println(filePath);
		File file = new File(filePath);
		HttpHeaders httpHeaders = new HttpHeaders();
		byte[] copyToByteArray = null;
		String downloadFielName ="";
		try {
			//下载显示的文件名，解决中文名称乱码问题  
			downloadFielName = new String(file.getName().getBytes("UTF-8"),"iso-8859-1");
	        //通知浏览器以attachment（下载方式）打开图片
	        httpHeaders.setContentDispositionFormData("attachment", downloadFielName); 
	        //application/octet-stream ： 二进制流数据（最常见的文件下载）。
	        httpHeaders.setContentType(MediaType.APPLICATION_OCTET_STREAM);
	        copyToByteArray = FileCopyUtils.copyToByteArray(file);
		}catch (Exception e) {
			e.printStackTrace();
		}
	return new ResponseEntity<byte[]>(copyToByteArray, httpHeaders, HttpStatus.CREATED);
	}
}
````

