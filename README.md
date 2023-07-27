# I. Bối cảnh
Bạn vừa được thuê làm Nhà phân tích dữ liệu Ecommerece cho Maven Factory, một nhà bán lẻ trực tuyến vừa tung ra sản phẩm đầu tiên của họ. Là một thành viên
của nhóm, bạn làm việc với Giám đốc điều hành, Trưởng phòng Tiếp thị và Người quản lý Trang web để giúp điều hành doanh nghiệp. Bạn sẽ phân tích và tối ưu hóa
các kênh tiếp thị, đo lường và kiểm tra hiệu suất chuyển đổi trang web và sử dụng dữ liệu để hiểu tác động của việc ra mắt sản phẩm mới

# II. Phân Tích
1. Business concept: Traffic source analysis (phân tích nguồn lưu lượng truy cập)
- phân tích nguồn lưu lượng truy cập là hiểu khách hàng của bạn đến từ đâu và kênh nào đang mang lại lưu lượng truy cập chất lượng cao nhất.
- Thông thường, khi doanh nghiệp triển khai chiến dịch marketing trả tiền, họ thường quan tâm đến hiệu suất và đo lường mọi thứ: từ số tiền
họ chi tiêu, đến hiệu quả chuyển đổi lưu lượng thành doanh số bán hàng và các chỉ số khác. Điều này đảm bảo rằng họ có thể đánh giá được sự thành công của chiến dịch và điều chỉnh chiến lược nếu cần

1.1 Tìm hiểu các nguồn lưu lượng lớn nhất

      import pandas as pd
      import matplotlib.pyplot as plt
      data = pd.read_csv('d:\SQL Maven\Website_sessions.csv', usecols=['utm_source','created_at', 'utm_campaign', 
                                                                       'http_referer', 'website_session_id'])
      data = data[(data['created_at'] < '2012-04-12')]
      data.groupby(['utm_sources','utm_campaign','http_referer']).agg({'website_session_id': 'nunique'}).reset_index()
      data= data.rename(columns={'website_session_id': 'sessions'})
      data.plot(kind='bar', x='utm_campaign', y='sessions')
      plt.xlabel('Campaign')
      plt.ylabel('Sessions')
      plt.title('Number of Sessions by Campaign')
      plt.show()
![image](https://github.com/IamQuangg/MavenFuzzyFactory/assets/128073066/a3695256-5f23-49e4-a6c1-7abd14c48be4)
![Number of sessions campaign](https://github.com/IamQuangg/MavenFuzzyFactory/assets/128073066/e6043b06-fbcd-473d-91d1-1bb868e31b01)

Ta có thể thấy rằng lượng truy cập được tìm kiếm thông qua ggsearch và không phải để quảng cáo thương hiệu có số lượng truy cập nhiều nhất.
Tuy nhiên chúng ta không chắc những lượng truy cập này là hiệu quả, để làm rõ hơn chúng ta cần xem xét lượng truy cập này mang lại doanh thu
là bao nhiêu dựa vào tỉ lệ chuyển đổi.

      import pandas as pd
      website_sessions = pd.read_csv("d:\SQL Maven\Website_sessions.csv")
      data = pd.merge(website_sessions, orders, on = 'website_session_id',how = 'left')
      data = data[(data['created_at_x'] < '2012-04-14')
            &(data['utm_source'] == 'gsearch')
            &(data['utm_campaign'] == 'nonbrand')]
      sessions = data['website_session_id'].nunique()
      orders = data['order_id'].count()
      session_to_order_conv_rt = (orders/sessions)*100
      data = pd.DataFrame({
      'sessions': [sessions],
      'orders': [orders],
      'session_to_order_conv_rt': [session_to_order_conv_rt]})
      data
![image](https://github.com/IamQuangg/MavenFuzzyFactory/assets/128073066/c608ffbe-7f25-41cd-b582-e5b4f895babe)

Sau khi xem xét ta có thể thấy rằng tỉ lệ chuyển dổi khá thấp (2,8%), điều này nghĩa là chúng ta nên giảm chi phí cho chiến dịch này vì chúng 
không hiệu quả. 

2. Business concept: Analyzing top website content (Phân tích nội dung trang web hàng đầu)
- Phân tích nội dung trang web để biết những trang web nào được xem nhiều nhất bởi người dùng, từ đó tìm ra web mà doanh nghiệp nên tập
trung vào để cải thiển hoạt động kinh doanh.

      import pandas as pd
      import matplotlib.pyplot as plt
      data= pd.read_csv('d:\SQL Maven\Website_Pageviews.csv')
      data['created_at'] = pd.to_datetime(data['created_at'])
      data = data[data['created_at']<'2012-06-09']
      data = data.groupby('pageview_url').agg(pageview=('website_session_id','nunique'))
      data=data.sort_values(by ='pageview', ascending= False)
      data = data.reset_index()
      data.plot(kind='bar', x='pageview_url', y='pageview',color = 'blue')
      plt.xlabel('pageview_url')
      plt.ylabel('pageview')  
      plt.title('Top Website Pages')
      plt.show()
![Top website pages](https://github.com/IamQuangg/MavenFuzzyFactory/assets/128073066/2d91e682-ade6-49f8-8b47-f99c9b574d39)

Có thể nhận thấy trang chủ "home" là trang được người dùng truy cập nhiều nhất trong tất cả các trang. Tuy nhiên ta sẽ cùng đi
sâu hơn để tìm hiểu tại sao những trang khác lại có tỉ lượng người dùng truy cập thấp mà trang "home" lại có lượng người truy cập cao như vậy


