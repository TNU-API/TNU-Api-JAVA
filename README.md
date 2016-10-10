# Hướng dẫn sử dụng TNU-Api-JAVA

Đầu tiên bạn import tất cả thư viện trong thư mục /bin vào dự án Java hoặc Android

Sau khi thêm, cập nhật lại project.

# Note:
Để có được AppId và AppSecret, bạn vui lòng liên hệ tran.trong@outlook.com

Sau đó sử dụng thư viện như ví dụ sau:
# CODE:
{

  import java.util.HashMap;
  import java.util.Map;

  import org.kingdark.api.ApiClient;
  import org.kingdark.api.ApiConnection;
  import org.kingdark.api.ApiConnectionCallback;
  import org.kingdark.api.ApiResponse;
  import org.kingdark.api.ApiSession;
  import org.kingdark.api.ApiVersion;
  import org.kingdark.api.exception.ApiCallbackNotFoundException;
  import org.kingdark.api.exception.ApiException;
  import org.kingdark.tnu.TnuApplication;

  public class run implements ApiConnectionCallback {
    private ApiClient cli;

    public static void main(String[] args) {
      new run();
    }

    public run() {

      // Tạo TnuApplication với app id và app secret để xác thực truy vấn
      TnuApplication app = new TnuApplication("APP-ID","APP-SECRET");

      // Tạo ApiClient có nhiệm vụ tạo và gửi truy vấn
      cli = new ApiClient(app);

      // FORM DATA gửi kèm theo, tại đây là ví dụ về form đăng nhập
      Map<String, String> signinData = new HashMap<>();
      signinData.put("username", "tester111");
      signinData.put("password", "tester111");

      // Dùng ApiClient để tạo ApiConnection với ApiRequest có method POST tới api signin kèm theo signinData
      ApiConnection conn = cli.post("signin", signinData);

      // Thiết lập callback cho ApiConnection với đối tượng this đã implement interface ApiConnectionCallback
      conn.setCallBack(this);

      try {
        // Kiểm tra phiên bản của Api bằng class ApiVersion
        System.err.println(ApiVersion.get());
        // Trả về mã phiên bản, tiện lợi cho việc so sánh
        System.err.println(ApiVersion.getId());

        // Thực thi ApiConnection để truy vấn tới server
        conn.execute();
      } catch (ApiCallbackNotFoundException e) { // Bắt ngoại lệ ApiConnectionCallback không hợp lệ
        e.printStackTrace();
      }
    }

    @Override
    public void onApiConnectSuccess(ApiResponse response) { // Truy vấn thành công, trả về ApiResponse. (ApiConnectionCallback)
      System.out.println("Thanh cong ===> " + response.getBody()); // dữ liệu trả về
      if ( response.getRequest().getPath().equalsIgnoreCase("signin") ) {
        ApiSession session = ApiSession.from(response); // lấy ApiSession từ dữ liệu đăng nhập trả về

        String sessionRaw = session.toString(); // ApiSession ở dạng lưu trữ ( sử dụng khi cần lưu trữ phiên người dùng và thiết bị )
        session = ApiSession.from(sessionRaw); // Chuyển từ dạng lưu trữ về ApiSession dùng được ( sử dựng khi cần lấy phiên đã lưu )

        if ( cli != null ) {
            cli.setSession(session); // Đưa session vào ApiClient để truy vấn theo phiên người dùng. Lúc này, các truy vấn sau sẽ có thêm thông tin phiên đăng nhập của người sử dụng
        }

      }
    }

    @Override
    public void onApiConnectFail(ApiException exception) { // Client gặp sự cố, không thể truy vấn. (ApiConnectionCallback)
      System.err.println("Loi Client ==> " + exception.getClass());
    }

    @Override
    public void onApiConnectError(ApiResponse response) { // Truy vấn nhưng server trả lỗi. (ApiConnectionCallback)
      System.err.println("Loi Server => " + response.getResponseCode()); // In ra mã lỗi
      System.err.println("Loi Server => " + response.getRawBody()); // In ra thông tin lỗi
    }
  }

}
