# Samten Sound & Music Therapy · web app

Demo trải nghiệm số cho Samten, một trung tâm trị liệu âm thanh thật (Samten Space, Lầu G,
hotline 0918 939 980). App là cửa vào, buổi trọn vẹn 60 phút diễn ra ở phòng thật.

**Live:** https://samten-web.vercel.app
**Deploy:** `vercel deploy --prod --yes` từ thư mục gốc. Account `lethienkhiem`, project `samten-web`.

## Quy tắc bắt buộc

1. **Không dùng dấu em dash (`—`) ở bất kỳ đâu trong file**, kể cả trong comment code.
   Khách yêu cầu rõ. Dùng dấu `·` (middot) hoặc dấu phẩy. Kiểm bằng:
   `python -c "print(open('index.html',encoding='utf-8').read().count('—'))"` phải ra 0.
   (Hai dấu `—` trong chính mục 1 này là cố ý, vì đang nêu tên ký tự và viết lệnh đếm nó.
   Quy định áp cho `index.html` và `samten-intro.html`, không áp cho file này.)
2. **Không hứa hiệu quả y khoa.** Bằng chứng khoa học của sound therapy còn mỏng, và quảng cáo
   dịch vụ sức khoẻ ở Việt Nam bị siết chặt. Viết theo **cảm giác** ("khi đầu quá ồn", "khi tim
   đập nhanh"), không viết theo bệnh ("chữa lo âu", "giảm trầm cảm").
3. **Không bịa dữ kiện thật.** Xem mục "Còn thiếu" ở cuối.
4. **Đo, đừng đoán.** Xem mục "Cách kiểm chứng" ở cuối. Bài học đắt nhất của dự án này.

## Kiến trúc

Toàn bộ app là **một file `index.html` duy nhất**, khoảng 2800 dòng: HTML + `<style>` + `<script>`,
không build step, không dependency ngoài Google Fonts và ảnh Pexels. Mọi âm thanh **tổng hợp trực
tiếp bằng Web Audio API**, không có file nhạc nào. Đây là điểm khác biệt cốt lõi của sản phẩm.

### File trong repo

| File | Vai trò |
|---|---|
| `index.html` | Toàn bộ app |
| `samten-intro.html` | Bản preview riêng của trang giới thiệu, dùng để khách duyệt trước khi ghép. Đã ghép vào app rồi, giữ lại làm tham chiếu. Xem tại `/samten-intro` |
| `vercel.json` | `cleanUrls: true` (nên `/samten-intro.html` redirect 308 sang `/samten-intro`), 2 header bảo mật |
| `CLAUDE.md` | File này |
| `.claude/launch.json` | Cấu hình `preview_start` dựng `python -m http.server 8899` để đo bản local. Bắt buộc dùng server: mở bằng `file://` thì pane render ảnh tĩnh, JS không chạy nên không đo được gì |

### Điều hướng

Nav ba ô, Samten ở giữa là vòng đồng chữ S (`.nav-btn.mid`), **phẳng bằng hàng** với hai ô kia:

```
Thư viện  ·  (S) Samten  ·  Cài đặt
```

Nhưng có **năm** view. `bowl` và `mix` là **trang con**, không có ô nav riêng, vào từ Cài đặt và có
nút `.sub-back` ở đầu trang:

```js
const VIEWS  = ["lib", "samten", "set", "bowl", "mix"];
const NAV_OF = { lib:"lib", samten:"samten", set:"set", bowl:"set", mix:"set" };
goView(v)   // hàm điều hướng duy nhất, dùng cho cả nav, [data-go] và .sub-back
```

Lý do: khách muốn tiêu điểm dồn vào **danh mục**, không vào chuông rung. Trước đây nav có 4 ô
(Gõ chuông | Thư viện | Pha trộn | Samten) và vào app là vào Gõ chuông. Nay vào app là vào Thư viện.

### Năm danh mục

Khách chốt taxonomy này, **không tự đổi**. Định nghĩa ở `const CATS`:

| id | Nhãn Việt | Tên khách đặt (`en`) | Chấm màu |
|---|---|---|---|
| `sound` | Âm chuông | Sound | `#e0b978` |
| `medi` | Thiền | Meditation | `#b28ee4` |
| `music` | Nhạc thư giãn | Music Relaxation | `#7abec6` |
| `noise` | **Thiên nhiên** | Noise | `#8fbe93` |
| `inst` | Nhạc cụ | Instrument | `#e8ac8e` |

`noise` giữ nhãn Việt là "Thiên nhiên" vì nội dung là sóng, mưa, gió; "Tiếng ồn" trong tiếng Việt
mang nghĩa xấu. Tên English vẫn hiện đúng theo taxonomy.

Thư viện render **thành từng khối theo danh mục** (`renderGrid`), mỗi khối có chấm màu, nhãn Việt,
tên English và số bản. Chọn chip "Tất cả" thì hiện cả năm khối; chọn một chip thì chỉ còn khối đó.
12 track, phân bố 3/2/2/3/2.

## Hệ thiết kế

```
Dark:  nền #170b1d · --night #241130 · --night2 #30183e · --night3 #3f2050
       vầng sáng đỉnh #5c2464 · --cream #f5ebe6 · --muted #b49dc0
Light: nền #f0e9f0 · --night #faf5f8 · --night2 #ffffff · --night3 #f1e8ef
       --cream (chữ) #2c2030 · --muted #6f6278 · --brass #96652a
Chung: --brass #d2a566 · --brass-soft #eed4a6
```

Tương phản đã kiểm, mọi cặp đều vượt WCAG AA (thấp nhất 4.65:1). Font Roboto, tiêu đề dùng
`font-style:italic` cho từ nhấn. Dấu `·` là chữ ký nhịp xuyên suốt. Bo góc 14 đến 22px.

**View Samten (`#view-samten`) có CSS lồng hoàn toàn trong `#view-samten`.** Bắt buộc: `.eyebrow`
và `.cat` đã tồn tại ở nơi khác trong app, để selector trần sẽ đè lên nhãn danh mục trên thẻ nhạc
và tiêu đề mọi trang. Keyframes của view này đặt tên riêng: `smBob`, `smBreathe`, `smRipple`.

## Âm thanh

### Chuỗi gain

```
engine -> master (0.7 * VOL_MAX) -> analyser -> compressor -> makeup -> limiter -> destination
                                    threshold -12dB, ratio 3, knee 20
                                    makeup 2.0 · limiter -3dB ratio 20
const VOL_MAX = 1.25, MAKEUP = 2.0;
```

Đo bằng OfflineAudioContext: to hơn chuỗi gốc **+9.4 dB** (RMS 2.96 lần), và **không clip** ở mọi
kịch bản gồm cả 100% âm lượng với nhiều lớp mix cùng bật (đỉnh cao nhất 0.984).

### Quy tắc một nguồn

`stopAllAudio(keep)` là hàm tắt trung tâm, gọi từ **mọi** cửa vào: `goView`, `playTrack`,
`mixToggle` (khi bật), và cả bốn tính năng (Hành trình, Ngân cùng chuông, Vẽ thanh âm, Chế độ
không gian). `keep` nhận `"track" | "mix" | "journey" | "hum" | "draw" | "space" | null`.

Quy tắc: chỉ một nguồn sống tại một thời điểm. Track được giữ khi đi giữa Thư viện, Samten và
Cài đặt (mini player vẫn hiện, đó là bản người dùng tự chọn), nhưng bị tắt khi vào Gõ chuông
hoặc Pha trộn vì hai trang đó có không gian âm riêng.

### Các nguồn âm

`player.handle` (track) · `mixState[id].handle` (4 lớp mix) · `jr.layers` (hành trình) ·
`hum` (mic) · `dw` (vẽ) · `sp` (chế độ không gian) · `bin.handle` (nhịp não) · `sing.nodes`
(xoa vành chuông) · các cú gõ tức thời.

## Bẫy đã gặp · đọc trước khi sửa gì

Đây là phần giá trị nhất của file này. Mỗi mục là một bug thật đã mất nhiều lượt để tìm.

### CSS

**`overflow-x` trên `body` giết chết việc cuộn.** `html,body{overscroll-behavior-y:none;
overflow-x:hidden}` làm `overflow-y` của body tự tính thành `auto`, nên body thành scroll container
không có chỗ cuộn, mà `overscroll-behavior-y:none` lại cấm chuyền cú cuộn lên viewport. Kết quả:
wheel và vuốt tay đều đứng im, trong khi `window.scrollTo()` vẫn chạy. Đúng cách:

```css
html{overscroll-behavior-y:none;overflow-x:hidden}
body{overflow-x:clip}   /* clip cắt được mà KHÔNG tạo scroll container */
```

**`body{overflow-x:clip}` là bắt buộc, đừng bỏ.** Trang có 7 phần tử dùng inset âm (`-6%` đến
`-60%`: canvas chuông, `.aurora`, `.cer-glow`). Không có chỗ cắt thì layout rộng hơn màn hình,
khung lệch, trống bên phải, kéo ngang được.

**`#bowlStage{touch-action:none}` là ĐÚNG, đừng đổi sang `pan-y`.** Xoa vành là cử chỉ rê ngón theo
vòng tròn, phải giữ trọn quyền điều khiển. Tôi từng đổi sang `pan-y` kèm hai lớp zone và nó phá
cử chỉ xoa. Trang cuộn được nhờ sửa ở `body`, không nhờ nhường vùng cảm ứng của chuông.

**`touch-action` lấy GIAO của phần tử với mọi tổ tiên.** Con chỉ thắt được, không nới ra được.
Muốn chia vùng cử chỉ thì cha phải nới lỏng, con thắt lại, và dùng các lớp anh em chồng nhau.

**`#sheetBg` phải nằm NGOÀI `.pane`.** `.pane` là scroll container; con `position:absolute` của nó
cuộn theo nội dung rồi trượt khỏi tầm nhìn, để lộ nền trơn bên dưới thành hai tông màu bị cắt.
`position:fixed` cũng không cứu được vì `.pane` có `transform` nên thành containing block.
Cách đúng: `#sheetBg` là con của `.sheet`, `.pane` thành `background:transparent;z-index:1`.

**`.pane` dùng `height:100%`, không phải `100dvh`.** `100dvh` co lại khi thanh địa chỉ mobile hiện
ra, còn khung cha `position:fixed;inset:0` thì không, hở một dải 168px ở đáy.

**`radial-gradient(circle ...)` mặc định là `farthest-corner`**, mốc `%` ăn theo đường chéo chứ
không theo bán kính. Dùng `closest-side` để mốc `%` khớp bán kính, nhờ đó mọi lớp về alpha 0 trước
khi `border-radius` cắt tới. Không làm vậy sẽ hiện một cung sáng lệch tâm.

**Đừng transition `font-size`.** Đó là animate layout, mỗi frame một lần reflow, và nếu transition
bị ngắt giữa đường thì chữ kẹt ở cỡ trung gian.

**`animation` luôn thắng khai báo thường.** Nếu keyframes giữ `filter` thì rule `.hit{filter:...}`
sẽ không có tác dụng. Đã gặp ở `.cer-orb`: phải bỏ `filter` khỏi keyframes nhịp thở để cú loé lúc
chạm hoạt động được.

**`prefers-reduced-motion` có rule chặn toàn cục** ở `index.html` (`*{animation:none!important;
transition-duration:.01ms!important}`). Nó làm màn welcome trông như hỏng: đốm sáng tĩnh, chạm thì
loé rồi tắt phụt, không sóng, không mảnh vỡ. Đã thêm một block `@media` phía sau để trả lại fade
độ sáng và fade opacity cho `.cer-orb` (không phải chuyển động không gian nên không gây chóng mặt),
và `waveFx.still()` vẽ ba vòng đứng yên. **Nếu máy test báo `reduce` thì bạn sẽ không thấy hiệu ứng
nào, đừng kết luận là code sai.**

**Canh giữa bằng `vh` cố định là đoán, không phải đo.** Chế độ chìm đắm từng đẩy khối
ảnh + tên + đồng hồ xuống `translateY(7vh)`. Ở khung 375x812 thì gần giữa (trên 110px, dưới 144px),
nhưng ở 412x680 thì đồng hồ tụt sát đáy (trên 101px, dưới 32px) vì phần trên khối là chiều cao
tuyệt đối còn `7vh` co theo khung. Nay `zenCenter()` đo mốc thật rồi đặt biến `--zenY`; đo lại ở
360x600 (63/62), 412x680, 430x915 (223/223) đều ra khe trên bằng khe dưới. `7vh` chỉ còn là giá trị
dự phòng trong `var(--zenY,7vh)`. Ba chi tiết của phép đo này, thiếu cái nào cũng lệch:

1. **Phải tắt transition trước khi đo.** `getBoundingClientRect` khi transform đang chuyển trả về
   giá trị NỘI SUY, không phải mốc 0 vừa gán, nên phép đo lệch đúng bằng phần transition còn dở.
   Class `.sheet.measuring` tắt transition, gán 0, đọc rect, bỏ class, rồi mới gán giá trị thật.
2. **Mốc dưới là `#elapsed`, không phải `.np-time`,** và còn trừ `0.16em`: dòng "phát liền mạch"
   dưới chữ số đã mờ nhưng vẫn chiếm ~23px, còn `line-height:1` làm mép dưới line box nằm dưới chân
   chữ số thêm ~0.16em (Roboto).
3. **`--zenGap` dồn khoảng chết bằng transform.** `.np-desc` và `.np-use` mờ nhưng vẫn chiếm ~98px,
   đẩy đồng hồ xa tên bài (ở 360x600 cả khối chiếm 584/600px, không còn khe nào). Kéo đồng hồ lên
   bằng `translateY(calc(var(--zenY) - var(--zenGap)))` · dùng transform nên không có cú giật
   reflow như khi thu `max-height` hay `display:none`.

**Đáy màn là MỘT khối `#dock`, không phải hai khối rời.** `#dock` là phần tử fixed duy nhất ở đáy,
bên trong nó `.mini` (trong luồng, `margin:0 8px 8px`) ngồi ngay trên `<nav>` · bố cục kiểu Spotify.
Đo ở khung 412x680: dock cao 150px khi có mini, 79px khi không. `.view` lấy `padding-bottom:
calc(var(--dockH) + 22px)` với `--dockH` do `setDockH()` đo, gọi lại trong `updateUI` và khi resize.

Đường đi tới đây là hai lần sai, đừng đi lại:

1. Bản đầu để mini và nav là **hai khối fixed rời nhau**, mỗi khối tự canh với mép đáy. Vòng đồng
   nhô lên 20px thì chọc vào mini, mini lại có z-index cao hơn nên cắt ngang vòng thành một đường
   thẳng. Nới `bottom` của mini ra chỉ làm mini trôi lơ lửng giữa màn · khách nói thẳng là "quá xấu".
2. **`getBoundingClientRect` KHÔNG tính `box-shadow`.** Vòng đồng có vành cắt spread 7px và quầng
   vươn lên ~10px, nên mép NHÌN THẤY cao hơn mép hình học tới 17px. Đo hình học ra 99px rồi để mini
   ở 104px thì vẫn cắt. Khi canh khoảng cách với phần tử có shadow, cộng tay spread và
   `blur - offsetY`.

**Ba icon nav phải cùng CHIỀU CAO HỘP và cùng cỡ quang học, hai chuyện khác nhau.** Hộp: svg để
`height:28px` bằng vòng đồng, `preserveAspectRatio` mặc định canh giữa nét vẽ nên không méo · đo
được tâm cả ba đều ở cùng một y. Cỡ quang học: nét vẽ trong viewBox 24 mỗi icon một khác (lưới
chiếm 16 đơn vị, bánh răng chiếm 22.6), nên bánh răng dùng `viewBox="-3.4 -3.3 30 30"` để thu về
16.5px và `stroke-width:1.9` bù phần bị thu. Nav cũng dùng `grid` ba cột bằng nhau chứ không phải
`space-around` · space-around chia khoảng theo bề rộng từng ô nên ô giữa lệch khỏi tâm.

**Icon Cài đặt là bánh răng, không phải mặt trời.** Mặt trời/mặt trăng là nút đổi sáng tối ở
header; để mặt trời ở nav thì hai nơi cùng một icon mà khác chức năng.

**Nút nổi của trình duyệt trong ứng dụng đè lên giữa đáy, và trang không tắt được nó.** Samsung
Internet (và Zalo, Messenger) đậu một nút tròn hiện thanh công cụ ngay giữa đáy khung web, khoảng
50px đến 95px tính từ mép đáy · đúng dải hàng nav. Không có cách nào thoát: muốn tránh hẳn thì phải
bỏ trống 95px cuối màn hình. Vì vậy nav để phẳng, không có gì nhô ra (Spotify, YouTube Music cũng
chịu đúng chỗ đè này) · phần bị che là icon phẳng chứ không phải một vòng đồng bị chặt đầu.

### JavaScript

**Đừng đặt tên hàm trùng.** Đã có `makeCard()` (tạo thiệp tải về của Hành trình) ở cuối file.
Function declaration hoist nên bản khai báo sau ghi đè bản trước. Hàm tạo thẻ nhạc phải tên
`makeTrackCard`. Kiểm trùng tên:

```bash
python -c "import re;b=open('index.html',encoding='utf-8').read();from collections import Counter;print({k:v for k,v in Counter(re.findall(r'^function\s+(\w+)',b,re.M)).items() if v>1})"
```

**AudioContext trên mobile sinh ra ở trạng thái `suspended`, và `ctx.resume()` là async.** Gõ xong
phát ngay thì tiếng đầu tiên rơi mất. Dùng `whenAudible(fn)` để dồn tiếng tới đúng lúc context đã
`running`. PC thường đã `running` sẵn nên đi thẳng, không thêm độ trễ.

**Compressor không có makeup gain thì chỉ làm nhỏ đi.** Chuỗi gốc `threshold -20dB, ratio 6` không
có gain bù, nên nhạc nhỏ hơn nhiều so với volume máy.

**Loa điện thoại không phát được hạ âm.** Engine `drone` từng có bội 0.5 (27.5Hz ở root 55), không
loa máy nào phát được mà vẫn ngốn headroom và kéo limiter nén cả bài. Đã bỏ, thêm bội 3, 4, 6 và
highpass 38Hz: năng lượng trên 150Hz tăng **+14 dB**. Khi thêm engine trầm, luôn kiểm dải trên
150Hz, không chỉ kiểm RMS tổng.

**`setInterval` cập nhật đồng hồ tham chiếu `jr`, `hum`, `dw`, `sp` được khai báo SAU nó.** Bình
thường vô hại, nhưng nếu có lỗi làm script dừng giữa đường thì nó phun lỗi TDZ 2 lần mỗi giây và
che mất lỗi gốc. Chưa sửa. Nếu thấy loạt lỗi `Cannot access 'jr' before initialization` thì lỗi
thật nằm ở chỗ khác, tìm lỗi đầu tiên trong console.

**`killSing()` phải dừng và ngắt nodes**, không chỉ hạ gain. Bản cũ để oscillator chạy mãi.
`sing.ensure()` tự dựng lại khi xoa lần sau nên tháo hẳn là an toàn.

## Nội dung

Mỗi track có ba trường chữ: `name` (tên), `desc` (câu cảm giác), `use` (dòng liều lượng khô, hiện
trong trang chi tiết dưới dạng `.np-use`). Ví dụ:

```
Ba hồi chuông khuya
Chuông nhỏ, quãng nghỉ dài · dành cho phút cuối ngày.
KHI KHÓ VÀO GIẤC · 15-30 PHÚT · ĐỂ LOA NHỎ CẠNH GIƯỜNG
```

Mục **Nhịp não** có giải thích cơ chế và chống chỉ định thật (động kinh, co giật, lái xe). Đừng bỏ.

Mục **Về Samten** trong Cài đặt theo bố cục: Một buổi diễn ra thế nào → Trước khi tới → Nên hỏi
chúng tôi trước nếu bạn → Hệ sinh thái. Phần chống chỉ định viết là "nên hỏi trước" chứ không phải
"không được tham gia", vừa đúng thực hành vừa không dựng rào cản với khách.

## Còn thiếu · cần khách cung cấp

**Mục "Ai dẫn".** Có một HTML comment đánh dấu chỗ chèn trong phần Về Samten. Cần: tên người dẫn,
học hoặc được chứng nhận ở đâu, dẫn từ năm nào, đã dẫn khoảng bao nhiêu buổi. Đây là thứ quyết định
người ta có đặt buổi hay không, nhưng là dữ kiện thật về người thật, **không được tự viết**.

**Ảnh thật.** Toàn bộ ảnh đang là Pexels stock. Trang giới thiệu thương hiệu dùng ảnh stock thì
người xem nhận ra ngay. Cần ảnh chụp tại Samten Space.

**Trình tự buổi 60 phút.** Phần "Một buổi diễn ra thế nào" viết từ dữ kiện đã có trong app. Nếu
trình tự thật khác thì phải sửa.

## Cách kiểm chứng

Bài học đắt nhất của dự án: **`window.scrollTo()` không chứng minh được là cuộn được.** Nó đi thẳng
vào viewport và bỏ qua toàn bộ chuỗi mà cuộn-bằng-tay phải đi qua, nên nó vẫn chạy ngon trong khi
wheel và vuốt tay đều đứng im. Tôi đã báo "ổn" nhiều lượt dựa trên phép đo sai.

Cách đúng:

```
Cuộn:      resize_window preset mobile (emulate touch) rồi computer{action:"scroll"}
           tại toạ độ cụ thể, đọc scrollY trước và sau. So sánh A/B bằng cách
           dựng lại CSS cũ ngay trên trang rồi cuộn lại.
Âm thanh:  OfflineAudioContext, dựng lại cả hai chuỗi, so RMS và đỉnh.
           Kiểm riêng dải trên 150Hz cho các engine trầm.
Layout:    getBoundingClientRect, so tâm phần tử với tâm khung.
Vòng đời:  bọc hàm để đếm lời gọi thật, và snapshot trạng thái mọi nguồn âm
           tại từng bước của luồng người dùng.
Cú pháp:   trước mỗi deploy, tách <script> ra file tạm rồi node --check,
           và đếm cân bằng brace cùng comment trong <style>.
```

**Lưu ý về môi trường test:** Browser pane thường không compositing, nên `screenshot` hay timeout và
**transition/animation không tiến** (đọc ra giá trị đầu, không phải giá trị đích). Muốn đọc giá trị
cuối thì gọi `el.getAnimations().forEach(a => a.finish())` trước khi đo. Nhiều lần tôi tưởng code
sai trong khi chỉ là đồng hồ animation của pane đứng.
