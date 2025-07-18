# githubTIL macro

## Github TIL 매크로 사용법

클립보드에 복사한 내용을 github에 업로드할 수 있도록 커밋과 푸시까지 자동으로 처리하는 도구입니다.

---

## 기능

- 클립보드 내용을 마크다운 파일로 저장
- 새 파일 생성 또는 기존 파일에 내용 추가 가능
- `git add`, `commit`, `push`를 자동 실행

---

## 실행 방법

![macro.png](attachment:40511fd5-0d21-4f3b-a4ce-cad73fd8252d:macro.png)

1. 프로그램을 실행하면 GUI 창이 열린다.
2. 다음 항목을 입력한다.
    - **Github TIL 레포 경로**: 로컬 Github TIL 저장소 경로
    - **저장할 폴더 이름**: 저장할 하위 폴더명 (기존에 없는 폴더명을 입력하면 새 폴더가 생성됩니다.)
    - **파일 제목**: 새 파일 생성 시 제목
    - **파일 이름**: 기존 파일에 추가 시 파일명 (확장자 포함해서 작성할 것)
3. 파일 생성 방식을 선택한다.
    - 새 파일 생성
    - 기존 파일에 추가
4. 클립보드에 내용을 복사한 뒤, `TIL 업로드` 버튼을 누른다.

마크다운 파일이 생성되거나 기존 파일에 내용이 추가되며, 자동으로 커밋과 푸시가 진행된다.

---
```

import javax.swing.*;
import java.awt.*;
import java.awt.datatransfer.*;
import java.io.*;

public class githubTIL_marcro {
	public static void main(String[] args) {

		  SwingUtilities.invokeLater(() -> {
	            JFrame frame = new JFrame("TIL 업로더");
	            frame.setSize(500, 450);
	            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
	            frame.setLayout(new GridLayout(0, 1, 10, 10));

	            JTextField repoPathField = new JTextField("C:/Users/SSAFY/Desktop/TIL"); // 본인 경로
	            JTextField folderField = new JTextField("Java");
	            JTextField titleField = new JTextField("TIL");
	            JTextField fileNameField = new JTextField("파일 이름 입력 (기존 파일 추가용)");

	            JRadioButton newFileButton = new JRadioButton("새 파일 생성", true);
	            JRadioButton appendFileButton = new JRadioButton("기존 파일에 추가");
	            ButtonGroup group = new ButtonGroup();
	            group.add(newFileButton);
	            group.add(appendFileButton);

	            JButton uploadButton = new JButton("TIL 업로드 🚀");

	            frame.add(new JLabel("📂 Github TIL 레포 경로:"));
	            frame.add(repoPathField);
	            frame.add(new JLabel("🗂️ 저장할 폴더 이름:"));
	            frame.add(folderField);
	            frame.add(new JLabel("📝 파일 제목 (새 파일 생성시 사용):"));
	            frame.add(titleField);
	            frame.add(new JLabel("📄 파일 이름 (기존 파일 추가시 사용, 확장자 포함):"));
	            frame.add(fileNameField);

	            frame.add(newFileButton);
	            frame.add(appendFileButton);
	            frame.add(uploadButton);

	            uploadButton.addActionListener(e -> {
	                String repoPath = repoPathField.getText().trim();
	                String folderName = folderField.getText().trim();
	                String title = titleField.getText().trim();
	                String fileNameInput = fileNameField.getText().trim();
	                String clipboardContent = getClipboardContents();

	                if (clipboardContent.isEmpty()) {
	                    JOptionPane.showMessageDialog(frame, "클립보드에 내용이 없습니다.", "에러", JOptionPane.ERROR_MESSAGE);
	                    return;
	                }

	                String folderPath = repoPath + "/" + folderName;
	                new File(folderPath).mkdirs(); // 폴더 없으면 생성

	                String usedFileName;
	                if (newFileButton.isSelected()) {
	                    usedFileName = createMarkdownFile(folderPath, title, clipboardContent);
	                } else {
	                    if (fileNameInput.isEmpty()) {
	                        JOptionPane.showMessageDialog(frame, "기존 파일 이름을 입력하세요.", "에러", JOptionPane.ERROR_MESSAGE);
	                        return;
	                    }
	                    usedFileName = appendToFile(folderPath, fileNameInput, clipboardContent);
	                }

	                String commitMessage = "TIL 업데이트: " + usedFileName;
	                runGitCommands(repoPath, commitMessage);

	                JOptionPane.showMessageDialog(frame, "업로드 완료!", "성공", JOptionPane.INFORMATION_MESSAGE);
	            });

	            frame.setVisible(true);
	        });
	    }

	    public static String getClipboardContents() {
	        try {
	            Toolkit toolkit = Toolkit.getDefaultToolkit();
	            Clipboard clipboard = toolkit.getSystemClipboard();
	            Transferable contents = clipboard.getContents(null);
	            if (contents != null && contents.isDataFlavorSupported(DataFlavor.stringFlavor)) {
	                return (String) contents.getTransferData(DataFlavor.stringFlavor);
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	        return "";
	    }

	    public static String createMarkdownFile(String folderPath, String title, String content) {
	        String fileName = java.time.LocalDate.now() + "-" + title.replaceAll("\\\\s+", "-") + ".md";
	        String filePath = folderPath + "/" + fileName;

	        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filePath))) {
	            writer.write("# " + title + "\\n\\n");
	            writer.write(content);
	            writer.write("\\n\\n---\\n");
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        return fileName;
	    }

	    public static String appendToFile(String folderPath, String fileName, String content) {
	        String filePath = folderPath + "/" + fileName;

	        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filePath, true))) {
	            writer.write("\\n\\n---\\n\\n");
	            writer.write(content);
	            writer.write("\\n\\n");
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        return fileName;
	    }

	    public static void runGitCommands(String repoPath, String commitMessage) {
	        try {
	            ProcessBuilder builder = new ProcessBuilder();
	            builder.directory(new java.io.File(repoPath));
	            
	            builder.command("git", "pull", "origin", "main");
	            Process pullProcess = builder.start();
	            pullProcess.waitFor();

	            builder.command("git", "add", ".");
	            builder.start().waitFor();
	            
	            builder.command("git", "status");
	            Process process = builder.start();

	            // 표준 출력 읽기
	            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
	            String line;
	            while ((line = reader.readLine()) != null) {
	                System.out.println(line);
	            }

	            // 표준 에러 읽기
	            BufferedReader errorReader = new BufferedReader(new InputStreamReader(process.getErrorStream()));
	            while ((line = errorReader.readLine()) != null) {
	                System.err.println(line);
	            }

	            // 프로세스 종료 코드 출력
	            int exitCode = process.waitFor();
	            System.out.println("Exit code: " + exitCode);



	            builder.command("git", "commit", "-m", commitMessage);
	            builder.start().waitFor();
	            builder.command("git", "log", "-1", "--oneline");


	            builder.command("git", "push");
	            builder.start().waitFor();

	            System.out.println("Git push 완료!");
	        } catch (Exception e) {
	            e.printStackTrace();
	        }

	}

}

\n\n
