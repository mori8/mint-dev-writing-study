# [[react] s3 파일 다운로드 버튼 구현](https://velog.io/@kados22/react-s3-%ED%8C%8C%EC%9D%BC-%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C-%EB%B2%84%ED%8A%BC-%EA%B5%AC%ED%98%84)

완전히 새로운 서비스로 넘어가기 전, 자료 소유자들이 pdf로 자료를 다운받을 수 있도록 다운로드 버튼을 만들어달라는 요청을 받았다.

이미 기존에 자료 주문 번호를 가지고 s3에 올라가있는 자료 주소를 받아오는 커스텀 Hook 있었으므로 그것을 활용해서 클라이언트 사이드에서 파일을 다운받을 수 있도록 해야했다.


```Javascript
import React, { useState, useEffect } from 'react';
import { usePdfRead } from '@src/hooks/pdf';

export default function DownloadButton({ orderItemId }: T) {
	const [downloadable, setDownloadable] = useState(false);
	const { data, loading } = usePdfRead<T>(orderItemId);

  // pdfUrl이 로드되었는지 확인
	useEffect(() => {
		if (data && data.pdfUrl) setDownloadable(true);
	}, [data]);

	const handleDownload = () => {
		if (downloadable) {
			fetch(data.pdfUrl, { method: 'GET' })
				.then(res => {
					return res.blob();
				})
				.then(blob => {
					const url = window.URL.createObjectURL(blob);
					const a = document.createElement('a');
					a.href = url;
					a.download = `{원하는 파일명}.pdf`;
					document.body.appendChild(a);
					a.click();
					setTimeout(_ => {
						window.URL.revokeObjectURL(url);
					}, 60000);
					a.remove();
					setOpen(false);
				})
				.catch(err => {
					console.error('err: ', err);
				});
		} else {
			alert(
				'PDF 다운에 실패했습니다. 다시 한 번 시도해주세요. 지속적인 실패 시 문의부탁드립니다.',
			);
		}
	};

	return (
		<Button onClick={handleDownload}>
  <div>
						다운로드
</div>
</Button>
	);
}


```

참고한 주소
- [Download From Amazon Simple Storage Service(S3) Buckets using JavaScript](https://jun711.github.io/aws/download-from-amazon-s3-buckets-using-javascript/)
